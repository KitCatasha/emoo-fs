# EMOO-FS

### Adapting Ensemble Multi-Objective Optimisation for Omics Feature Selection

The original EMOO framework was developed for hyperparameter optimisation by Moradpour et al. (2026) [1]. In this project, I adapted the framework for feature selection.

In EMOO-FS, candidate solutions are represented as binary feature masks, where selected genes are encoded as 1 and excluded genes as 0. NSGA-II is used to search for feature subsets that optimise multiple objectives simultaneously: accuracy, specificity, sensitivity, F1-score, metric stability, and the number of selected features. The resulting subsets are evaluated using a Random Forest classifier.

> The source code for this project is maintained in a private repository. This public repository presents the methodology, experimental design, results, and selected visualisations.

---

## Motivation

Omics datasets often contain far more features than samples. In gene-expression data, thousands of genes may be measured for a comparatively small number of patients, creating a high-dimensional feature-selection problem.

Many features may be redundant or provide little predictive information, while retaining too many features can increase computational cost, overfitting, and difficulty in biological interpretation. Feature selection is therefore particularly important in omics-based machine learning.

Multi-objective optimisation provides a way to consider several competing objectives simultaneously. Rather than optimising only one metric, methods such as NSGA-II can search for solutions that balance predictive performance and feature-subset size.

NSGA-II produces a set of non-dominated solutions known as the **Pareto front**, where each solution represents a different trade-off between the optimisation objectives.

---

## Aim

The aim of this project was to adapt the existing EMOO framework from hyperparameter optimisation to feature selection and evaluate its performance against baseline feature-selection methods and the single-solution multi-objective approach MOOF.

The project also examined:

- whether using all Pareto-optimal solutions improves performance
- whether selecting only the top-ranked Pareto solutions is more effective
- whether different ensemble strategies influence classification performance
- how different pre-filter sizes affect performance and selected feature subsets

---

## Dataset

The project used the **TCGA Breast Cancer (TCGA-BRCA)** gene-expression dataset.

The dataset contained:

- **20,531 genes**
- **1,069 patients**
- **494 Luminal A samples**
- **575 samples from other breast-cancer subtypes**

The classification task was to distinguish **Luminal A** from other molecular subtypes.

This dataset represents a typical high-dimensional omics setting, where the number of features is much larger than the number of samples.

---

## Methodology

Because direct evolutionary search over more than 20,000 genes was infeasible, a pre-filtering step was applied before multi-objective feature selection.

The dataset was split into 80% training and 20% test data. All preprocessing and feature-selection steps were fitted only on the training set to prevent data leakage.

Preprocessing included:

- CPM normalisation
- log₂(CPM + 1) transformation
- variance filtering
- Welch’s t-test
- Benjamini-Hochberg FDR correction
- mutual-information ranking

Four candidate feature spaces were evaluated:

- Top 500 genes
- Top 300 genes
- Top 200 genes
- Top 100 genes

Each candidate solution was encoded as a binary feature mask. A Random Forest classifier was then trained and evaluated using only the selected genes.

### Optimisation objectives

**Maximised**
- Accuracy
- Specificity
- Sensitivity
- F1-score

**Minimised**
- Standard deviation across classification metrics
- Number of selected features

---

## Workflow

![EMOO-FS workflow](figures/emoo_fs_workflow.png)

---

## Methods Compared

To evaluate EMOO-FS, I compared it with two baseline feature-selection methods and a single-solution multi-objective approach.

| Method | Type | Selection strategy |
|---|---|---|
| **t-test top-k** | Statistical baseline | Selects half of the pre-filtered genes using the smallest FDR-adjusted p-values |
| **RFECV-RF** | Model-based baseline | Recursive feature elimination with cross-validation |
| **MOOF** | Multi-objective, single solution | NSGA-II followed by TOPSIS |
| **EMOO-FS** | Multi-objective ensemble | NSGA-II followed by ensemble learning |

MOOF uses a similar multi-objective optimisation approach to EMOO-FS, but reduces the Pareto front to a single final solution using TOPSIS. EMOO-FS instead combines Pareto-optimal solutions through ensemble learning.

---

## Experimental Setup

Feature subsets were optimised using NSGA-II with:

- **100 generations**
- **Population size:** 50
- **Crossover probability:** 0.95
- **Mutation probability:** 0.05
- **Cross-validation:** Repeated Stratified 5-fold × 3
- **Base classifier:** Random Forest
- **Evaluation:** 5 random seeds

Due to the computational cost of the evolutionary search, the experiments were run on the Lise High-Performance Computing (HPC) system.

---
