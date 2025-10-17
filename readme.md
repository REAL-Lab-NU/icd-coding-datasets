# Medical Coding Datasets Documentation

## Table of Contents
- [Abstract](#abstract)
- [Datasets Overview](#datasets-overview)
- [CodiEsp Dataset](#codiesp-dataset)
- [MIMIC-III Dataset](#mimic-iii-dataset)
- [MIMIC-IV Dataset](#mimic-iv-dataset)
- [Dataset Comparison](#dataset-comparison)
- [Data Access and Preparation](#data-access-and-preparation)
- [Citations](#citations)

---

## Abstract

This document provides comprehensive information about the medical coding datasets used in this project. Medical coding is the task of automatically assigning diagnosis and procedure codes (ICD-9, ICD-10) to clinical narratives from electronic health records. This project utilizes three major datasets: **CodiEsp** (Spanish clinical cases), **MIMIC-III** (US critical care records), and **MIMIC-IV** (updated US critical care records).

### Key Challenges in Medical Coding
- **Large Label Space**: ICD-10 contains 70,000+ diagnostic codes
- **Long Clinical Documents**: Discharge summaries can exceed 4,000 tokens
- **Multi-label Classification**: Each clinical case may have 10+ diagnosis codes
- **Imbalanced Distribution**: Some codes appear rarely in training data
- **Domain Expertise**: Requires deep medical knowledge and clinical reasoning

---

## Datasets Overview

| Dataset | Language | Coding System | Time Period | # Cases | # Codes | Document Type | Avg. Codes per Case |
|---------|----------|---------------|-------------|---------|---------|---------------|---------------------|
| **CodiEsp** | Spanish (→English) | ICD-10 | N/A | 1,000 (test) | ~16,000 | Clinical case reports | ~7.5 |
| **MIMIC-III** | English | ICD-9 | 2001-2012 | 52,722 | 8,929 (full) | Discharge summaries | ~15.9 |
| **MIMIC-IV** | English | ICD-9 / ICD-10 | 2008-2019 | 331,794 | 15,000+ | Discharge summaries | ~12.3 |

---

## CodiEsp Dataset

### Overview

The **CodiEsp (Clinical Coding in Spanish) Dataset** is a gold standard corpus of Spanish clinical cases coded in ICD-10, created for the CLEF eHealth 2020 shared task.

### Dataset Characteristics

| Characteristic | Details |
|----------------|---------|
| **Source** | Spanish clinical case reports from SciELO |
| **Language** | Spanish (original) → English (translated via GPT-3.5) |
| **Coding System** | ICD-10 (CIE-10 in Spanish) |
| **Total Cases** | 1,000 clinical cases (test set) |
| **Document Type** | Full clinical case reports with diagnoses and procedures |
| **Average Length** | Varies (clinical case narratives) |
| **Annotation** | Manually coded by medical professionals |
| **ICD-10 Codes** | ~16,000 unique diagnosis codes |
| **Codes per Case** | Average ~7.5 codes per clinical case |

### Key Features

- **Clinical Case Format**: Unlike discharge summaries, these are structured clinical case presentations
- **Spanish Medical Terminology**: Original documents use Spanish medical vocabulary
- **Translation Required**: LLM approaches typically translate to English first
- **ICD-10 Only**: Uses the international ICD-10 coding system (not ICD-9)
- **Evaluation Metrics**: Macro/Micro-averaged Precision, Recall, F1-score

### Tree-Search Algorithm Context

The CodiEsp dataset is commonly used with **LLM-guided tree-search algorithms** that exploit the hierarchical structure of ICD-10:

1. **Hierarchical ICD-10**: Parent codes represent broader conditions, child codes are specific
2. **Search Process**: Start at root → LLM selects branches → iterate until terminal nodes
3. **Efficiency**: Avoids evaluating all 70,000+ codes exhaustively

### Data Access

**Download Link**: [CodiEsp Dataset on Zenodo](https://zenodo.org/records/3837305#.XsZFoXUzZpg)

**GitHub Repository**: [https://github.com/anand-subu/automated-clinical-coding-llm](https://github.com/anand-subu/automated-clinical-coding-llm)

### Data Preparation Steps

```bash
# 1. Clone the CodiEsp processing repository
git clone https://github.com/anand-subu/automated-clinical-coding-llm.git
cd automated-clinical-coding-llm

# 2. Download CodiEsp dataset from Zenodo
wget https://zenodo.org/records/3837305/files/codiesp.zip

# 3. Extract dataset
unzip codiesp.zip
# This creates: final_dataset_v4_to_publish/ directory with:
#   - Clinical case files (.txt)
#   - Gold standard TSV files with ICD-10 annotations

# 4. (Optional) Translate Spanish cases to English using GPT-3.5
python translate_files.py \
    --input_dir final_dataset_v4_to_publish/test/text_files \
    --output_dir translated_test_set

# 5. Run ICD coding with tree-search
python run_tree_search.py \
    --input_dir translated_test_set \
    --output_file predictions.json \
    --model_name gpt-3.5-turbo-0613

# 6. Evaluate performance
python evaluate_performance.py \
    --input_json predictions.json \
    --gold_standard_tsv final_dataset_v4_to_publish/test/codiesp_test.tsv
```

---

## MIMIC-III Dataset

### Overview

**MIMIC-III (Medical Information Mart for Intensive Care III)** is a large, freely-available database comprising deidentified health-related data from patients admitted to critical care units at Beth Israel Deaconess Medical Center.

### Dataset Characteristics

| Characteristic | Details |
|----------------|---------|
| **Source** | Beth Israel Deaconess Medical Center (Boston, USA) |
| **Patient Population** | Critical care (ICU) patients |
| **Time Period** | 2001-2012 |
| **Version** | v1.4 (used in this project) |
| **Total Admissions** | 58,976 hospital admissions |
| **Unique Patients** | 46,520 patients |
| **Document Type** | Discharge summaries (clinical narratives) |
| **Coding System** | ICD-9-CM (diagnosis) and ICD-9-PCS (procedures) |
| **Language** | English |

### MIMIC-III Splits

This project uses multiple benchmark splits:

#### 1. MIMIC-III Full
- **Cases**: 52,722 discharge summaries
- **ICD-9 Codes**: 8,929 unique codes
- **Avg. Codes/Case**: ~15.9
- **Source**: [Explainable Prediction of Medical Codes (Mullenbach et al., NAACL 2018)](https://aclanthology.org/N18-1100/)

#### 2. MIMIC-III 50
- **Cases**: 8,067 discharge summaries
- **ICD-9 Codes**: 50 most frequent codes
- **Purpose**: Simplified benchmark for faster experimentation
- **Source**: Same as MIMIC-III Full

#### 3. MIMIC-III Clean (from this project)
- **Cases**: Similar to Full, with improved preprocessing
- **Improvements**: Corrected code assignment, deduplication
- **Purpose**: Address data quality issues found in original splits

### Key Features

- **Comprehensive Clinical Data**: Includes diagnoses, procedures, medications, lab results
- **Discharge Summaries**: Detailed clinical narratives written by physicians
- **ICD-9 Codes**: Both diagnosis codes (ICD-9-CM) and procedure codes (ICD-9-PCS)
- **Research Standard**: Widely used benchmark in automated medical coding research
- **Imbalanced Distribution**: Long-tail distribution of code frequencies

### Data Access

**Official Portal**: [MIMIC-III v1.4 on PhysioNet](https://physionet.org/content/mimiciii/1.4/)

**GitHub Repository**: [https://github.com/JoakimEdin/medical-coding-reproducibility](https://github.com/JoakimEdin/medical-coding-reproducibility)

**Access Requirements**:
1. **Complete CITI Training**: Free online course (2-3 hours)
   - Human Subjects Research
   - HIPAA and Privacy Protection
2. **Sign Data Use Agreement**: Commit to responsible data usage
3. **Request Access**: Submit credentialed access request
4. **Approval**: Typically granted within 1-2 days

### Data Preparation Steps

```bash
# 1. Clone the medical coding reproducibility repository
git clone https://github.com/JoakimEdin/medical-coding-reproducibility.git
cd medical-coding-reproducibility

# 2. Download MIMIC-III v1.4 from PhysioNet
# Visit: https://physionet.org/content/mimiciii/1.4/
# After approval, download and extract to a directory (e.g., ~/data/mimiciii)

# 3. Configure path in settings
# Edit src/settings.py and set:
# DOWNLOAD_DIRECTORY_MIMICIII = "~/data/mimiciii"  # Your download path

# 4. Install dependencies
pip install -e .

# 5. Prepare MIMIC-III Full/50 splits (Mullenbach benchmark)
python prepare_data/prepare_mimiciii_mullenbach.py
# Output: files/data/mimiciii_full/ and files/data/mimiciii_50/

# 6. Prepare MIMIC-III Clean splits (recommended, improved preprocessing)
python prepare_data/prepare_mimiciii.py
# Output: files/data/mimiciii_clean/
```

---

## MIMIC-IV Dataset

### Overview

**MIMIC-IV** is the latest version of the MIMIC database, covering a more recent time period and including both ICD-9 and ICD-10 codes due to the US healthcare transition in October 2015.

### Dataset Characteristics

| Characteristic | Details |
|----------------|---------|
| **Source** | Beth Israel Deaconess Medical Center (Boston, USA) |
| **Patient Population** | Critical care (ICU) and general hospital patients |
| **Time Period** | 2008-2019 |
| **Version** | v2.2 (used in this project) |
| **Total Admissions** | 431,231 hospital admissions |
| **Unique Patients** | 299,712 patients |
| **Document Type** | Discharge summaries from clinical notes |
| **Coding System** | **ICD-9** (2008-2015) and **ICD-10** (2015-2019) |
| **Language** | English |

### MIMIC-IV Splits

#### 1. MIMIC-IV ICD-9
- **Cases**: ~165,000 discharge summaries (2008-2015)
- **ICD-9 Codes**: ~15,000 unique codes
- **Avg. Codes/Case**: ~12.3
- **Purpose**: Benchmark for ICD-9 coding, comparable to MIMIC-III

#### 2. MIMIC-IV ICD-10
- **Cases**: ~166,000 discharge summaries (2015-2019)
- **ICD-10 Codes**: ~15,000 unique codes
- **Avg. Codes/Case**: ~12.3
- **Purpose**: Modern benchmark for ICD-10 coding (used in this project)

### Key Features

- **Larger Scale**: ~6x more admissions than MIMIC-III
- **Modern Coding**: Includes ICD-10 transition period
- **Improved Data Quality**: Enhanced preprocessing and validation
- **Dual Coding Systems**: Enables ICD-9 to ICD-10 transfer learning research
- **Longer Documents**: Discharge summaries average ~1,500 tokens

### Data Access

**Official Portals**:
- **MIMIC-IV v2.2**: [PhysioNet MIMIC-IV](https://physionet.org/content/mimiciv/2.2/)
- **MIMIC-IV-NOTE v2.2**: [PhysioNet MIMIC-IV-NOTE](https://physionet.org/content/mimic-iv-note/2.2/)

**GitHub Repository**: [https://github.com/JoakimEdin/medical-coding-reproducibility](https://github.com/JoakimEdin/medical-coding-reproducibility)

**Access Requirements**:
- Same as MIMIC-III (CITI training + Data Use Agreement)
- Must request access to **both** MIMIC-IV and MIMIC-IV-NOTE

### Data Preparation Steps

```bash
# 1. Clone the medical coding reproducibility repository (if not already done)
git clone https://github.com/JoakimEdin/medical-coding-reproducibility.git
cd medical-coding-reproducibility

# 2. Download MIMIC-IV and MIMIC-IV-NOTE from PhysioNet
# Visit and download:
#   - MIMIC-IV v2.2: https://physionet.org/content/mimiciv/2.2/
#   - MIMIC-IV-NOTE v2.2: https://physionet.org/content/mimic-iv-note/2.2/
# Extract to directories (e.g., ~/data/mimiciv and ~/data/mimiciv-note)

# 3. Configure paths in settings
# Edit src/settings.py and set:
# DOWNLOAD_DIRECTORY_MIMICIV = "~/data/mimiciv"
# DOWNLOAD_DIRECTORY_MIMICIV_NOTE = "~/data/mimiciv-note"

# 4. Install dependencies (if not already done)
pip install -e .

# 5. Prepare MIMIC-IV splits (both ICD-9 and ICD-10)
python prepare_data/prepare_mimiciv.py
# Output:
#   - files/data/mimiciv_icd9/    (ICD-9 data: 2008-2015)
#   - files/data/mimiciv_icd10/   (ICD-10 data: 2015-2019)
```

### Current Project Usage

This project focuses on **MIMIC-IV ICD-10** for the following reasons:
- **Modern Relevance**: ICD-10 is the current global standard
- **Larger Code Space**: More challenging task (70,000+ codes vs 15,000)
- **Real-world Applicability**: Matches current clinical coding practices
- **Evidence Extraction**: GPT-4o used to extract clinical evidence supporting each code

**Generated Files** (in `files/data/mimiciv_icd10/`):
- `mimiciv_icd10.feather`: Structured discharge summaries with ICD-10 codes
- `structured_discharge_summaries.jsonl`: Parsed sections (history, hospital_course, etc.)
- `evidence_top3_20k.jsonl`: GPT-4o extracted evidence for top-3 codes per case
- `icd10_code_descriptions.jsonl`: ICD-10 code to description mapping
- Train/validation/test splits

---

## Dataset Comparison

### Document Length Statistics

| Dataset | Avg. Tokens | Median Tokens | 95th Percentile | Max Tokens |
|---------|-------------|---------------|-----------------|------------|
| CodiEsp | ~800 | ~700 | ~1,400 | ~2,000 |
| MIMIC-III Full | ~1,500 | ~1,200 | ~3,500 | ~10,000 |
| MIMIC-IV ICD-10 | ~1,500 | ~1,300 | ~3,800 | ~10,200 |

### Code Distribution Characteristics

| Dataset | Total Codes | Codes ≥ 10 samples | Codes ≥ 100 samples | Distribution |
|---------|-------------|-------------------|---------------------|--------------|
| CodiEsp | ~16,000 | ~2,000 | ~500 | Long-tail |
| MIMIC-III Full | 8,929 | 3,426 | 923 | Long-tail |
| MIMIC-IV ICD-10 | ~15,000 | ~4,500 | ~1,200 | Long-tail |

### Task Complexity

| Aspect | CodiEsp | MIMIC-III | MIMIC-IV ICD-10 |
|--------|---------|-----------|-----------------|
| **Label Space Size** | Very Large (16K) | Large (9K) | Very Large (15K) |
| **Document Length** | Medium | Long | Long |
| **Language** | Spanish→English | English | English |
| **Code Hierarchy** | ICD-10 (deep) | ICD-9 (shallow) | ICD-10 (deep) |
| **Clinical Domain** | General cases | Critical care | Critical care + General |
| **Primary Challenge** | Cross-lingual, Tree search | Multi-label extreme classification | Long documents, Large label space |

---

## Data Access and Preparation

### Quick Start Guide

#### For CodiEsp Dataset

**Step 1: Clone Repository**
```bash
git clone https://github.com/anand-subu/automated-clinical-coding-llm.git
cd automated-clinical-coding-llm
```

**Step 2: Download Dataset**
- Visit: [Zenodo CodiEsp page](https://zenodo.org/records/3837305#.XsZFoXUzZpg)
- Download `codiesp.zip` (no credentials required)
- Extract in repository directory

**Step 3: Process Data**
- (Optional) Translate Spanish to English using `translate_files.py`
- Run tree-search ICD coding with `run_tree_search.py`
- Evaluate with `evaluate_performance.py`

**Documentation**: See repository `README.md` for detailed instructions

---

#### For MIMIC-III Dataset

**Step 1: Get Data Access**
1. Complete [CITI Human Subjects Research Training](https://about.citiprogram.org/) (free, 2-3 hours)
2. Create [PhysioNet](https://physionet.org/) account
3. Complete credentialing with signed Data Use Agreement
4. Request access to [MIMIC-III v1.4](https://physionet.org/content/mimiciii/1.4/)
5. Wait for approval (typically 1-2 days)

**Step 2: Clone Repository**
```bash
git clone https://github.com/JoakimEdin/medical-coding-reproducibility.git
cd medical-coding-reproducibility
```

**Step 3: Download MIMIC-III**
- After PhysioNet approval, download MIMIC-III v1.4
- Compressed: ~6 GB, Uncompressed: ~46 GB
- Extract to a local directory

**Step 4: Configure and Process**
```bash
# Install dependencies
pip install -e .

# Edit src/settings.py to set DOWNLOAD_DIRECTORY_MIMICIII

# Process data
python prepare_data/prepare_mimiciii_mullenbach.py  # For Full/50 splits
python prepare_data/prepare_mimiciii.py             # For Clean splits
```

**Documentation**: See repository `README.md` for detailed instructions

---

#### For MIMIC-IV Dataset

**Step 1: Get Data Access**
- Same as MIMIC-III (CITI training + PhysioNet credentialing)
- Request access to **both**:
  - [MIMIC-IV v2.2](https://physionet.org/content/mimiciv/2.2/)
  - [MIMIC-IV-NOTE v2.2](https://physionet.org/content/mimic-iv-note/2.2/)

**Step 2: Clone Repository** (if not already done)
```bash
git clone https://github.com/JoakimEdin/medical-coding-reproducibility.git
cd medical-coding-reproducibility
```

**Step 3: Download MIMIC-IV**
- After PhysioNet approval, download both datasets:
  - MIMIC-IV: Core clinical data (compressed: ~8 GB)
  - MIMIC-IV-NOTE: Clinical notes (compressed: ~4 GB)
- Extract to separate local directories

**Step 4: Configure and Process**
```bash
# Install dependencies (if not already done)
pip install -e .

# Edit src/settings.py to set:
#   DOWNLOAD_DIRECTORY_MIMICIV
#   DOWNLOAD_DIRECTORY_MIMICIV_NOTE

# Process data (creates both ICD-9 and ICD-10 splits)
python prepare_data/prepare_mimiciv.py
```

**Output Files**:
- `files/data/mimiciv_icd9/`: ICD-9 data (2008-2015)
- `files/data/mimiciv_icd10/`: ICD-10 data (2015-2019)

**Documentation**: See repository `README.md` for detailed instructions

---

## Citations

### CodiEsp Dataset

```bibtex
@inproceedings{miranda2020overview,
  title={Overview of automatic clinical coding: annotations, guidelines, and solutions for non-english clinical cases at codiesp track of CLEF eHealth 2020},
  author={Miranda-Escalada, Antonio and Gonzalez-Agirre, Aitor and Armengol-Estapé, Jordi and Krallinger, Martin},
  booktitle={Working Notes of Conference and Labs of the Evaluation (CLEF) Forum. CEUR Workshop Proceedings},
  year={2020}
}

@dataset{miranda_escalada_2020_3837305,
  author       = {Miranda-Escalada, Antonio and
                  Gonzalez-Agirre, Aitor and
                  Krallinger, Martin},
  title        = {{CodiEsp corpus: gold standard Spanish clinical
                   cases coded in ICD10 (CIE10) - eHealth CLEF2020}},
  month        = may,
  year         = 2020,
  publisher    = {Zenodo},
  version      = {1.4},
  doi          = {10.5281/zenodo.3837305},
  url          = {https://doi.org/10.5281/zenodo.3837305}
}
```

### MIMIC-III Dataset

```bibtex
@article{johnson2016mimic,
  title={MIMIC-III, a freely accessible critical care database},
  author={Johnson, Alistair EW and Pollard, Tom J and Shen, Lu and Lehman, Li-wei H and Feng, Mengling and Ghassemi, Mohammad and Moody, Benjamin and Szolovits, Peter and Anthony Celi, Leo and Mark, Roger G},
  journal={Scientific data},
  volume={3},
  number={1},
  pages={1--9},
  year={2016},
  publisher={Nature Publishing Group}
}
```

### MIMIC-IV Dataset

```bibtex
@article{johnson2023mimic,
  title={MIMIC-IV, a freely accessible electronic health record dataset},
  author={Johnson, Alistair EW and Bulgarelli, Lucas and Shen, Lu and Gayles, Alvin and Shammout, Ayad and Horng, Steven and Pollard, Tom J and Hao, Sicheng and Moody, Benjamin and Gow, Brian and others},
  journal={Scientific data},
  volume={10},
  number={1},
  pages={1},
  year={2023},
  publisher={Nature Publishing Group}
}
```

### Automated Medical Coding on MIMIC (This Project)

```bibtex
@inproceedings{edin2023automated,
  address = {Taipei, Taiwan},
  title = {Automated {Medical} {Coding} on {MIMIC}-{III} and {MIMIC}-{IV}: {A} {Critical} {Review} and {Replicability} {Study}},
  isbn = {978-1-4503-9408-6},
  shorttitle = {Automated {Medical} {Coding} on {MIMIC}-{III} and {MIMIC}-{IV}},
  doi = {10.1145/3539618.3591918},
  booktitle = {Proceedings of the 46th {International} {ACM} {SIGIR} {Conference} on {Research} and {Development} in {Information} {Retrieval}},
  publisher = {ACM Press},
  author = {Edin, Joakim and Junge, Alexander and Havtorn, Jakob D. and Borgholt, Lasse and Maistro, Maria and Ruotsalo, Tuukka and Maaløe, Lars},
  year = {2023}
}
```

### LLM-guided Tree Search for ICD Coding

```bibtex
@inproceedings{boyle2023automated,
  title={Automated clinical coding using off-the-shelf large language models},
  author={Joseph Boyle and Antanas Kascenas and Pat Lok and Maria Liakata and Alison O'Neil},
  booktitle={Deep Generative Models for Health Workshop NeurIPS 2023},
  year={2023},
  url={https://openreview.net/forum?id=mqnR8rGWkn}
}
```

---

## Additional Resources

### Official Documentation
- **PhysioNet**: [https://physionet.org/](https://physionet.org/)
- **MIMIC-III Documentation**: [https://mimic.mit.edu/docs/iii/](https://mimic.mit.edu/docs/iii/)
- **MIMIC-IV Documentation**: [https://mimic.mit.edu/docs/iv/](https://mimic.mit.edu/docs/iv/)
- **CITI Training**: [https://about.citiprogram.org/](https://about.citiprogram.org/)

### GitHub Repositories
- **CodiEsp Processing**: [https://github.com/anand-subu/automated-clinical-coding-llm](https://github.com/anand-subu/automated-clinical-coding-llm)
- **MIMIC Processing**: [https://github.com/JoakimEdin/medical-coding-reproducibility](https://github.com/JoakimEdin/medical-coding-reproducibility)

### Project File Structure
```
medical-coding-reproducibility/
├── files/data/
│   ├── mimiciii_full/          # MIMIC-III Full benchmark
│   ├── mimiciii_50/            # MIMIC-III 50 benchmark
│   ├── mimiciii_clean/         # MIMIC-III Clean (improved)
│   ├── mimiciv_icd9/           # MIMIC-IV ICD-9 (2008-2015)
│   └── mimiciv_icd10/          # MIMIC-IV ICD-10 (2015-2019)
├── prepare_data/
│   ├── prepare_mimiciii.py           # Process MIMIC-III Clean
│   ├── prepare_mimiciii_mullenbach.py # Process MIMIC-III Full/50
│   ├── prepare_mimiciv.py            # Process MIMIC-IV
│   └── utils.py                      # Shared utilities
└── src/
    └── settings.py                   # Configure data paths
```

---

**Last Updated**: October 2025

**Maintained by**: Medical Coding Reproducibility Project Team
