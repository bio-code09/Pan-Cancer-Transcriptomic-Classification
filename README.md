# Pan-Cancer Classification & Feature Selection

This project is a machine learning pipeline that identifies the most important genetic biomarkers to classify 5 different types of cancer (Breast, Lung, Colon, Kidney, and Prostate) using RNA-Seq data. 

Since genomic datasets are incredibly massive and noisy (over 20,000 genes per patient), the goal here was to find a way to strip away the noise without losing the accuracy of our diagnostic models.

---

## 📁 Project Structure

* **`data/`** - Contains `pancan_top100.arff`, the filtered version of the dataset containing only the top 100 genes selected by our model.
* **`scripts/`** - Contains `data_preprocessing_and_mapping.ipynb`, the Python script used to merge the raw files and track down the gene indexes.
* **`results/`** - Contains the raw WEKA execution text reports and threshold graphs.

---

## 💻 Project Workflow

To successfully process the genomic data and build the diagnostic models, the project followed this systematic pipeline:

* **Step 1: Data Merging & Preprocessing (Python)**
  * Real-world genomic data often separates actual measurements from patient diagnostics. Using a Python script in Google Colab, the raw RNA-Seq feature matrices (`raw.csv`) were mapped and integrated with categorical tumor labels (`labels.csv`) into a single master file. 
  * Missing elements and formatting anomalies were checked to ensure clean data input for machine learning software.

* **Step 2: Resolving RAM Bottlenecks (Environment Config)**
  * Because the complete dataset contains more than 20,000 distinct genes per sample, the default WEKA configuration immediately crashed due to memory limits.
  * To bypass this, the underlying `RunWeka.ini` configuration file was updated to change the Java environment options (`javaOpts`), expanding the memory allocation cap to 4GB (`-Xmx4g`). This modification stabilized the system to process high-dimensional calculations.

* **Step 3: Entropy-Based Feature Selection**
  * Training models on 20,532 attributes runs a high risk of model overfitting and slow speeds. To strip out the data noise, a non-parametric Information Gain filter (`InfoGainAttributeEval`) was run inside WEKA.
  * This evaluated and ranked every single gene based on how much clear information it provided about the specific cancer types.
  * A Ranking threshold was set to capture only the top 100 highest-yielding genes, completely reducing data dimensionality by 99.5%.

* **Step 4: Machine Learning Benchmarking**
  * The optimized 100-gene dataset was evaluated across two distinct structural algorithms: Random Forest (an ensemble tree-based classifier) and SMO (a support vector machine using geometric linear hyperplanes).
  * Validation was carried out using **10-Fold Stratified Cross-Validation** to guarantee that every test partition had an identical, balanced distribution of cancer types.

---

## 📊 Performance & Results

Both architectures delivered near-perfect diagnostics on the filtered genomic signatures, demonstrating that the 100 selected genes held incredibly strong biological signals.

| Evaluation Metric | 🌲 Random Forest (500 Trees) | 📐 Support Vector Machine (SMO) |
| :--- | :---: | :---: |
| **Correctly Classified Samples** | **99.75%** (799 / 801) | **99.37%** (796 / 801) |
| **Incorrectly Classified Samples** | **0.25%** (2 / 801) | **0.62%** (5 / 801) |
| **Kappa Statistic** | **0.9967** | **0.9918** |
| **Weighted Average F-Measure** | **0.998** | **0.994** |
| **Weighted Average ROC Area** | **1.000** | **1.000** |

### 🔍 Error Breakdown & Boundary Analysis

While the models were highly effective, a granular audit of the cross-validation confusion matrices highlights subtle challenges at the biological boundary lines:

* **Random Forest** experienced minimal confusion, misclassifying exactly 1 Lung Cancer (`LUAD`) sample as Breast Cancer (`BRCA`) and 1 Breast Cancer sample as Lung Cancer.
* **SMO (SVM)** showed slightly more boundary strain, misclassifying 3 Lung Cancer (`LUAD`) samples as Breast Cancer, 1 Breast Cancer sample as Lung Cancer, and 1 Kidney Cancer (`KIRC`) sample as Breast Cancer.

**Takeaway:** The consistent overlap between `LUAD` (Lung) and `BRCA` (Breast) samples shows that even after feature selection anonymizes the attributes, these two tumor types share overlapping transcriptomic mutations or cellular pathway behaviors. This slight cross-contamination is a signature of biological similarity rather than a mistake made by the algorithms.
