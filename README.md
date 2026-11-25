<p align="center"><img src="assets/lsma-lab-kidney-multi-omics-banner.svg" alt="LSMA Analysis Lab: kidney cancer, three layers at once" width="100%"></p>

# Kidney cancer, three layers at once: a multi-omics analysis in the LSMA Analysis Lab

**When gene activity, DNA methylation and mutations are measured in the same tumours, what do they say together, and does combining them tell us more than each one alone?**

This repository is a worked example from the **LSMA Analysis Lab**, the analysis platform of the [London School of Multiomics & AI](https://lsma.co.uk). The full analysis was run in the lab, without writing code: import of three public data layers for the same patients, how far the layers agree, links between layers, shared factors (MOFA), patient groups from all layers at once (similarity network fusion) with survival, a supervised model (DIABLO), and a comparison of all methods. The report, figures and tables it produced are here, with a robustness check and a comparison with known biology.

| | |
|---|---|
| **Data** | TCGA-KIRC (The Cancer Genome Atlas, clear cell kidney cancer): RNA-seq, DNA methylation (450K) and somatic mutations of the same tumours, from the NCI Genomic Data Commons |
| **Patients** | 100 primary tumours with all three layers (the lab imports at most 100 methylation arrays from TCGA) |
| **Layers** | expression (2,000 most variable genes), methylation (2,000 most variable CpG probes, after removing the sex chromosomes), mutations (32 genes mutated in at least 5 patients) |
| **Outcomes** | stage, vital status and survival time (30 deaths) |
| **Lab module** | Multi-omics integration (workflow version 1.0.0) |
| **Lab run time** | **5 min 41 s** for the whole analysis (importing and combining the three TCGA layers: about 4 min) |
| **Validation** | A robustness re-run with half the features (3 min 41 s), the published biology of ccRCC, and the companion [mutation analysis](https://github.com/MultiOmics-Intelligence/lsma-lab-somatic-mutations-kidney-cancer-tcga-kirc) |
| **Result** | Two patient groups with very different survival (log-rank p = 0.0006), reproduced with half the features. Combining the layers did not beat RNA alone, and the analysis led to a fix in the lab (see below) |

---

## Biological question and aim

**Question.** The companion repositories look at one molecular layer at a time. Here three layers of the same 100 tumours are analysed together. Do the layers agree about which patients are alike? Is there a shared pattern that runs through gene activity and methylation, and does it relate to how patients fare? And does integration add anything over the best single layer?

**Hypothesis.**
- Expression and methylation should agree strongly, because methylation helps switch genes on and off.
- A shared pattern should separate aggressive from less aggressive tumours, as reported by the TCGA consortium: higher stage, more methylation changes and worse survival.
- Mutations in *BAP1* should mark the more aggressive tumours, and *PBRM1* mutations the less aggressive.

**Aims**
1. Bring the three layers of the same patients together in the lab and check how far they agree.
2. Find links between single genes, methylation sites and mutations.
3. Find shared factors (MOFA) and patient groups (similarity network fusion), and relate them to stage and survival.
4. Test whether the layers can predict which patients died (DIABLO), with honest cross-validation.
5. Compare integration with each layer alone.
6. Check the results with fewer features and against known biology.

---

## Background: why combine layers?

Each layer tells a different part of the story:
- **Gene expression** (RNA) shows what the cell is doing now.
- **DNA methylation** marks the DNA in a way that keeps genes on or off for long periods.
- **Mutations** are the permanent DNA changes that drove the cancer.

**Multi-omics integration** asks what the layers say together. The lab offers several methods, because they ask different questions:
- **MOFA** finds hidden *factors*: patterns of variation, each shared by some or all of the layers.
- **Similarity network fusion (SNF)** builds a network of similar patients from each layer, fuses the networks and splits the patients into groups.
- **DIABLO** is told a known grouping (here, patients who died vs those alive) and looks for a small set of features in each layer that predicts it.

**Integration is not automatically better.** A single strong layer can carry most of the signal, and the lab's comparison stage is built to show that honestly.

---

## Key findings

1. **Expression and methylation agree; mutations stand apart** (Figure QC5). The distances between patients correlate at Mantel r = 0.53 for expression and methylation (p = 0.002), but only 0.04–0.07 between mutations and the other layers. Mutations are sparse: most tumours carry only a few of the 32 recurrent genes.
2. **One shared factor follows outcome** (Figures 1–3).
   - **MOFA found 6 factors, 5 shared between layers.** They explain 43% of the variation in expression and 37% in methylation, but little of the sparse mutation layer.
   - **Factor 1 relates to death and to stage** (R² = 0.22 with vital status, 0.16 with stage). It is lower in patients who died (mean −1.19 vs +0.51) and falls step by step from stage I to stage IV.
   - **Its genes are kidney metabolism genes**, high in the less aggressive tumours: *HAO2*, *CYP4A11*, *GLYATL1*, *ACADL*, *SLC27A2*, *AGTR1*, *NPR3*. At its other end are genes of aggressive disease such as *GFPT2*, *LAMB3* and *BASP1*. This resembles the known split of ccRCC into a better (ccA) and a worse (ccB) subtype.
3. **Two patient groups with very different survival** (Figure 4).
   - SNF divided the patients into two groups, 66 and 34 patients.
   - **The smaller group did much worse:** 18 of 34 patients died (53%), against 12 of 66 (18%); log-rank p = 0.0006. It also holds more stage III and IV tumours (17 of 29 staged patients, against 12 of 58).
4. **The layers predict death better than chance, but not well** (Figures 7–9).
   - **DIABLO's balanced error is 28%.** With shuffled labels it is 49% (permutation p = 0.02).
   - **Its mutation features point to the expected genes.** *BAP1*, *ZFHX4*, *MUC16* and *KMT2C* mutations carry weight towards death. *BAP1* and *ZFHX4* were also the genes most enriched in patients who died in the [mutation analysis](https://github.com/MultiOmics-Intelligence/lsma-lab-somatic-mutations-kidney-cancer-tcga-kirc).
5. **Integration did not beat RNA alone** (Figure 10). Asked to split the patients into two groups, RNA alone matches vital status best (adjusted Rand index 0.18). SNF comes close (0.17), and MOFA and canonical correlation follow (0.16), while pasting the layers together does worse (0.14). The lab says so in a warning. This is an honest and common result: in ccRCC, much of the prognostic signal is already in gene expression.
6. **Links between layers** (Figures 12–13).
   - 376 of 10,000 tested gene–methylation pairs correlate (adjusted p < 0.05).
   - *PBRM1* mutation goes with higher expression of the kidney tubule genes *ENPP3*, *SCGN* and *SLC17A3* (adjusted p = 0.002), consistent with PBRM1-mutant tumours being the less aggressive, more differentiated type.

Evidence labels from the lab: layer agreement, associations, survival and the DIABLO error are *statistically supported*; factors and groups are *exploratory*; their biological reading is *interpretation*.

---

## A fix in the lab, found by this analysis

The first run of this analysis gave a result that looked strong but was wrong. MOFA's first factor was the patients' **sex** (R² = 0.98), and the patient groups followed sex too.
- **The cause:** the methylation layer kept CpG probes on the X and Y chromosomes. 1,824 of the 5,000 most variable probes were on the sex chromosomes, and they differ between women and men far more than any tumour biology. The lab's own methylation workflow removes them in its quality control, but the multi-omics module didn't.
- **A second bug:** the DIABLO stage stopped with an error. In cross-validation, a gene mutated in only a few patients can have no carrier left among the training patients of a fold.

Both were fixed in the lab's code, and the results in this repository come from the corrected run:
- The multi-omics module now removes sex-chromosome probes from methylation layers (a new setting, on by default).
- DIABLO now drops such genes within a fold instead of stopping.

The fix is now live in the lab (25 November 2026), so students get the same corrected behaviour. The before-and-after comparison is in the Validation section. This is the purpose of the series: running real data from start to finish finds the weak points, and the fixes reach every student.

---

## Quality control and PCA

| Check | Result |
|---|---|
| Patients | 200 with RNA and mutations, 100 with methylation; **100 in every layer** (Figure QC2) |
| Features | expression 5,000 → 2,000 most variable genes; methylation 5,000 → 1,824 sex-chromosome probes removed → 2,000 most variable; mutations 154 genes → 32 mutated in at least 5 and absent in at least 5 patients (Figure QC1) |
| Missing values | none: the combined layers contain only features measured in every patient |
| Clinical data | stage missing for 13 patients; survival time and vital status complete |

**PCA of each layer (Figures QC3–QC4).**
- **Methylation:** PC1 (17% of all variation) moves from stage I tumours on one side to stage III and IV tumours on the other, the methylation–stage link reported by TCGA. The blue points without a stage label are the 13 patients with stage missing.
- **Expression:** shown the same way in Figure QC3.

| | |
|---|---|
| ![PCA methylation](figures/qc-pca-pca_methylation.png) **QC4.** PCA of the methylation layer, coloured by stage. | ![Layer agreement](figures/qc-pca-layer_agreement.png) **QC5.** Do the layers agree about the patients? Mantel correlation of patient distances. |
| ![Features per layer](figures/qc-pca-features_per_layer.png) **QC1.** Features in each layer, in the file and used. | ![PCA expression](figures/qc-pca-pca_expression.png) **QC3.** PCA of the expression layer, coloured by stage. |

Patients per layer (QC2) are in [`figures/`](figures).

---

## Figures

| | |
|---|---|
| ![Factors and clinical](figures/02-factors_and_sample_sheet.png) **Figure 2.** What the MOFA factors go with: factor 1 with vital status and stage. | ![Survival](figures/04-survival_by_group.png) **Figure 4.** Survival of the two SNF patient groups (log-rank p = 0.0006). |
| ![Factor 1 weights](figures/03-weights_factor1.png) **Figure 3.** The features behind factor 1 in each layer. | ![Shuffled labels](figures/08-shuffled_labels.png) **Figure 8.** DIABLO with real labels (dashed line) against shuffled labels (bars). |
| ![Against known](figures/10-against_known.png) **Figure 10.** Which method recovers vital status best? RNA alone. | ![Variance per layer](figures/01-variance_per_layer.png) **Figure 1.** How much of each layer the factors capture. |

All figures are in [`figures/`](figures), as PNG (300 dpi) and vector PDF:

| File | What it shows |
|---|---|
| 01 | Variance of each layer captured by the MOFA factors |
| 02 | MOFA factors and the clinical data |
| 03 | Features behind factor 1 |
| 04 | Survival of the SNF patient groups |
| 05 | Patient map of the fused network |
| 06 | How close each layer alone comes to the fused groups |
| 07 | DIABLO cross-validated error |
| 08 | DIABLO against shuffled labels |
| 09 | DIABLO features on component 1 |
| 10 | Which method recovers vital status best |
| 11 | How far the methods agree with each other |
| 12 | Gene–methylation correlations |
| 13 | Strongest mutation–expression links |
| qc-pca-* | Layers, patients, PCA per layer and layer agreement |

## The full report

The lab's own report covers every stage: what was done, why, the checks, the results and their limits. It's available as a [**PDF**](report/LSMA-Analysis-Lab-report-kidney-cancer-multi-omics.pdf) or as [**HTML**](report/LSMA-Analysis-Lab-report-kidney-cancer-multi-omics.html) (download it and open it in a browser for the interactive version).

## Summary tables ([`results/`](results))

| File | Contents |
|---|---|
| `layer_summary.tsv` | Each layer: patients, features in the file, features used, sex-chromosome probes removed |
| `layer_agreement_mantel.tsv` | Mantel correlation between the layers |
| `cross_layer_feature_pairs.tsv` | Gene–methylation correlations and mutation–feature tests, with adjusted p-values |
| `mofa_factor_values.tsv` | Each patient's value on each MOFA factor |
| `mofa_factors_and_clinical.tsv` | How strongly each factor relates to each clinical column |
| `mofa_top_weights.tsv` | The features with the largest weights on each factor |
| `snf_patient_groups.tsv` | Each patient's SNF group, and the group from each layer alone |
| `snf_groups_and_clinical.tsv` | SNF groups against vital status, stage, sex and diagnosis |
| `diablo_selected_features.tsv` | The features DIABLO chose in each layer |
| `diablo_agreement_of_layers.tsv` | How far the layers' DIABLO components agree |
| `methods_compared.tsv` | Every method's grouping against pasting the layers together and against vital status |

---

## Workflow

```mermaid
flowchart LR
    A1["TCGA-KIRC RNA-seq<br/>200 tumours"] --> C["Combine in the lab<br/>100 patients in every layer"]
    A2["TCGA-KIRC methylation<br/>100 tumours"] --> C
    A3["TCGA-KIRC mutations<br/>200 tumours"] --> C
    C --> I["Input<br/>remove X/Y probes<br/>most variable features"]
    I --> E["Exploration<br/>PCA per layer<br/>Mantel agreement"]
    I --> S["Associations<br/>between layers"]
    I --> F["MOFA2<br/>shared factors"]
    I --> K["SNF<br/>patient groups<br/>+ survival"]
    I --> D["DIABLO<br/>predict vital status<br/>+ permutations"]
    I --> P["Comparison<br/>every method vs<br/>each layer alone"]
    E --> H["Report, figures,<br/>tables"]
    S --> H
    F --> H
    K --> H
    D --> H
    P --> H
    F --> V1["Robustness<br/>1,000 features"]
    K --> V1
    D --> V2["Known biology<br/>BAP1, PBRM1, stage<br/>mutation repository"]
    V1 --> H
    V2 --> H
    classDef data fill:#F6F2EA,stroke:#B83A16,color:#0F1B33
    classDef step fill:#0F1B33,stroke:#0F1B33,color:#F4F4F2
    classDef check fill:#2F7D5B,stroke:#2F7D5B,color:#F4F4F2
    classDef out fill:#FBEAE3,stroke:#D9481F,color:#0F1B33
    class A1,A2,A3 data
    class C,I,E,S,F,K,D,P step
    class V1,V2 check
    class H out
```

## How it was done

Every step below is a stage in the LSMA Analysis Lab, run with the settings in [`settings/analysis-settings.json`](settings/analysis-settings.json).

1. **Import.**
   - The mutation data of 200 TCGA-KIRC tumours were imported first (from the [mutation analysis](https://github.com/MultiOmics-Intelligence/lsma-lab-somatic-mutations-kidney-cancer-tcga-kirc)).
   - RNA-seq and methylation were then imported asking for **the same patients**, as the lab allows for multi-omics. The lab imports up to 100 methylation arrays from TCGA.
2. **Combine.** The lab combined the three datasets into one multi-omics dataset, linking them by patient. It used:
   - expression as log2 counts per million (genes above 1 CPM in 20% of samples, the 5,000 most variable);
   - methylation as beta values (the 5,000 most variable probes);
   - mutations as 1 or 0 per gene (genes mutated in at least 3 patients).
3. **Input.** Patients in every layer were kept, and sex-chromosome probes removed from the methylation layer. Then the 2,000 most variable features of each continuous layer were kept, and mutated genes with at least 5 carriers and 5 non-carriers.
4. **Exploration.** PCA of each layer, and the Mantel correlation (Spearman, permutation p-value) between the patient distances of each pair of layers.
5. **Associations.** Spearman correlation between the 100 most variable genes and methylation probes, and the Wilcoxon test between mutated genes and the continuous layers. Benjamini–Hochberg correction was applied within each pair of layers.
6. **Factors.** MOFA2 with up to 8 factors, keeping those explaining at least 1% of variation, and their association with the clinical columns.
7. **Clustering.** Similarity network fusion (20 neighbours, σ = 0.5, 20 iterations), spectral clustering with the number of groups by the eigengap rule, and a log-rank test of survival between groups.
8. **Supervised.** DIABLO (block sparse PLS-DA) for vital status: 2 components, 15 features per layer and component, 5-fold cross-validation repeated 10 times, and 50 shuffles of the labels.
9. **Comparison.** The same patients grouped by each layer alone, by pasting the layers together, by SNF, by MOFA factors and by generalised canonical correlation (RGCCA). Each grouping is compared with the others and with vital status.

**Software:** R 4.5.2 and Bioconductor 3.21, with MOFA2 (mofapy2), SNFtool, mixOmics, cluster and survival, in the versions listed in the settings file.

---

## Validation

### 1. Before and after the lab fix
The same data were run before and after the fix to the multi-omics module ([`validation/before_and_after_the_lab_fix.tsv`](validation/before_and_after_the_lab_fix.tsv); the earlier factor heatmap is in [`validation/before-fix-factors_and_sample_sheet.png`](validation/before-fix-factors_and_sample_sheet.png)):

| Check | Before (sex-chromosome probes kept) | After (removed) |
|---|---|---|
| MOFA factor 1, R² with sex | **0.98** | 0.02 |
| MOFA factor 1, R² with vital status | 0.00 | **0.22** |
| SNF groups vs sex (adjusted Rand index) | 0.30 | −0.01 |
| Number of SNF groups | 4 | 2 |
| DIABLO stage | failed | completed |

### 2. Robustness: half the features
The analysis was run again in the lab keeping 1,000 instead of 2,000 features per continuous layer (3 min 41 s; [`validation/robustness_1000_features.tsv`](validation/robustness_1000_features.tsv)):

| Measure | 2,000 features | 1,000 features |
|---|---|---|
| MOFA factor 1 (correlation between the runs) | | **r = 0.99** |
| MOFA factor 2 (correlation between the runs) | | **r = 0.99** |
| SNF groups (eigengap rule) | 2 | 3 (the 2-group split, with the larger group divided; adjusted Rand index 0.55) |
| Survival difference between SNF groups | p = 0.0006 | p = 0.00006 |
| DIABLO balanced error (permutation p) | 0.28 (0.02) | 0.28 (0.02) |

The factors and the survival difference are stable. The exact number of groups is not, as the lab's low silhouette width (0.05) warns: the patients form a continuum more than separate types.

### 3. Known biology
- ***BAP1* towards death, *PBRM1* towards survival.** BAP1 loss marks aggressive ccRCC and PBRM1 loss the less aggressive form (TCGA 2013; Kapur et al. 2013). Here DIABLO gives *BAP1* a strong weight towards death, and *PBRM1* mutation goes with the more differentiated tubule genes.
- **The same genes as the mutation analysis.** *BAP1* and *ZFHX4* were the genes most enriched in patients who died in the [mutation analysis](https://github.com/MultiOmics-Intelligence/lsma-lab-somatic-mutations-kidney-cancer-tcga-kirc) (200 tumours). DIABLO picks both, from mutations alone.
- **Methylation follows stage.** The TCGA consortium reported more methylation changes in higher-stage tumours. Here methylation PC1 separates stage III and IV tumours from stage I (Figure QC4).
- **A metabolic factor.** Factor 1 is carried by kidney metabolism genes that aggressive tumours lose. This fits the metabolic shift found in the [proteomics](https://github.com/MultiOmics-Intelligence/lsma-lab-proteomics-kidney-cancer-cptac) and [metabolomics](https://github.com/MultiOmics-Intelligence/lsma-lab-metabolomics-kidney-cancer) analyses.

![Validation](validation/validation-fix-robustness-and-known-biology.png)

**Figure V1.**
- **Left:** before and after the lab fix.
- **Middle:** MOFA factor 1 with 2,000 and with 1,000 features per layer, coloured by vital status.
- **Right:** the mutations DIABLO links to death or survival.

**Verdict: robust for the factors and the survival difference; integration adds little over RNA alone.**

---

## Limitations

- **100 patients.** The lab imports at most 100 TCGA methylation arrays, so the integration uses 100 of the 200 tumours. Groups and factors from 100 patients are less stable than from several hundred, and there was no independent cohort with all three layers to repeat the analysis in.
- **Vital status is a crude outcome.** "Died vs alive at last follow-up" ignores how long each patient was followed. The survival test of the SNF groups uses the follow-up time; DIABLO does not.
- **Sex genes remain in the expression layer.** Only methylation probes on the sex chromosomes are removed. Genes such as *XIST* and *RPS4Y1* still vary with sex, and MOFA factor 5 relates to sex (R² = 0.19).
- **Mutations add little to the integrated methods.** With 32 sparse genes, the mutation layer contributes under 1% of the variation the factors capture. Its signal shows mainly in DIABLO.
- **Groups are a continuum.** The silhouette width of 0.05 says the patients do not fall into sharply separate groups. Two or three groups are equally defensible.
- **A teaching example, not clinical advice.** These results are for education and must not be used to diagnose or treat anyone.

---

## Repository structure

```
lsma-lab-multi-omics-kidney-cancer-tcga-kirc/
├── README.md                  this case study
├── LICENSE                    CC BY 4.0
├── assets/
│   └── lsma-lab-kidney-multi-omics-banner.svg   animated banner
├── report/                    the LSMA Analysis Lab's full report
│   ├── LSMA-Analysis-Lab-report-kidney-cancer-multi-omics.pdf
│   └── LSMA-Analysis-Lab-report-kidney-cancer-multi-omics.html
├── figures/                   key figures, PNG (300 dpi) and vector PDF
│   ├── 01-variance_per_layer ... 13-strongest_expression_mutations
│   └── qc-pca-*                                layers, patients, PCA, agreement
├── results/                   summary tables (tab-separated)
│   ├── layer_summary.tsv, layer_agreement_mantel.tsv, cross_layer_feature_pairs.tsv
│   ├── mofa_factor_values.tsv, mofa_factors_and_clinical.tsv, mofa_top_weights.tsv
│   ├── snf_patient_groups.tsv, snf_groups_and_clinical.tsv
│   ├── diablo_selected_features.tsv, diablo_agreement_of_layers.tsv
│   └── methods_compared.tsv
├── validation/                the lab fix, robustness and known biology
│   ├── validation-fix-robustness-and-known-biology.png / .pdf
│   ├── before-fix-factors_and_sample_sheet.png
│   ├── before_and_after_the_lab_fix.tsv
│   └── robustness_1000_features.tsv
└── settings/
    └── analysis-settings.json          every parameter and software version
```

## Why we share this analysis

This is one of a series of worked examples from the **LSMA Analysis Lab**, published openly for five reasons:

- **To show what the lab does, on real data.** Each repository takes one public dataset through one lab module, from import to report, and shows every step, figure and setting. Nothing is hidden or polished after the fact: the report here is the one the lab produced.
- **To teach a complete analysis, not just a result.** Students see the parts that are usually skipped: quality control, the choice of statistics, an independent validation, and an honest list of limitations.
- **To earn trust by checking known biology first.** These analyses deliberately reproduce findings that are already well established, such as the hypoxia programme and the Warburg switch in kidney cancer. A method that gets the known answers right can be trusted with new questions.
- **To keep testing our own tools.** Every example is a real-world test of the lab. Running public data from start to finish shows what works and what needs fixing, and the fixes go back into the lab for every student. Improvements from this series so far: a more reliable import from NCBI GEO, automatic choice of the right cell-type markers for each tissue in single-cell analysis, and, from this analysis, removal of sex-chromosome probes and a more robust DIABLO in multi-omics.
- **To make the work open and repeatable.** The data are public, every parameter and software version is listed, and diploma students can repeat the same analysis in the lab themselves.

The series follows one cancer, clear cell kidney cancer, through every layer: genes, single cells, DNA methylation, enhancers, mutations, proteins, metabolites and lipids, here all at once, and then in a [machine learning analysis](https://github.com/MultiOmics-Intelligence/lsma-lab-machine-learning-kidney-cancer-tcga-kirc) that predicts survival and stage. Each repository links to the others.

These examples are for education and must not be used for medical decisions.

## Data source and credits

- **Data:** The Cancer Genome Atlas (TCGA-KIRC), open-access RNA-seq counts, 450K methylation beta values and masked somatic mutations from the [NCI Genomic Data Commons](https://portal.gdc.cancer.gov/projects/TCGA-KIRC), under the [GDC data use policies](https://gdc.cancer.gov/access-data/data-access-processes-and-tools). No raw data are copied into this repository.
- **Reference studies:**
  - Cancer Genome Atlas Research Network. Comprehensive molecular characterization of clear cell renal cell carcinoma. *Nature* 2013;499:43–49.
  - Kapur P, Peña-Llopis S, Christie A, et al. Effects on survival of BAP1 and PBRM1 mutations in sporadic clear-cell renal-cell carcinoma: a retrospective analysis with independent validation. *Lancet Oncology* 2013;14:159–167.
- **Methods:**
  - Argelaguet et al., *Molecular Systems Biology* 2018 and *Genome Biology* 2020 (MOFA, MOFA+)
  - Wang et al., *Nature Methods* 2014 (similarity network fusion)
  - Singh et al., *Bioinformatics* 2019 (DIABLO)
  - Rohart et al., *PLoS Computational Biology* 2017 (mixOmics)
- **Companion analyses** (same cancer):
  - [bulk RNA-seq](https://github.com/MultiOmics-Intelligence/lsma-lab-bulk-rnaseq-kidney-cancer-tcga-kirc)
  - [single-cell RNA-seq](https://github.com/MultiOmics-Intelligence/lsma-lab-single-cell-kidney-cancer-ccrcc)
  - [DNA methylation](https://github.com/MultiOmics-Intelligence/lsma-lab-dna-methylation-kidney-cancer-tcga-kirc)
  - [active enhancers](https://github.com/MultiOmics-Intelligence/lsma-lab-chip-seq-kidney-cancer-enhancers)
  - [somatic mutations](https://github.com/MultiOmics-Intelligence/lsma-lab-somatic-mutations-kidney-cancer-tcga-kirc)
  - [proteomics](https://github.com/MultiOmics-Intelligence/lsma-lab-proteomics-kidney-cancer-cptac)
  - [metabolomics](https://github.com/MultiOmics-Intelligence/lsma-lab-metabolomics-kidney-cancer)
  - [lipidomics](https://github.com/MultiOmics-Intelligence/lsma-lab-lipidomics-kidney-cancer)
  - [machine learning](https://github.com/MultiOmics-Intelligence/lsma-lab-machine-learning-kidney-cancer-tcga-kirc)

## Run it yourself

Diploma students of the London School of Multiomics & AI can repeat this analysis in the **LSMA Analysis Lab**:
1. **Public data** → TCGA → *TCGA-KIRC*: import mutations, then RNA-seq and methylation asking for the same patients.
2. **Combine** the three datasets into one multi-omics dataset.
3. Run **Multi-omics** with survival (`survival_days`, event `vital_status` = Dead) and DIABLO on `vital_status`.

Find out more about the diplomas at [lsma.co.uk](https://lsma.co.uk).

## Licence

The text, figures and tables in this repository are shared under [Creative Commons Attribution 4.0 (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/): you may reuse them with credit to the London School of Multiomics & AI. TCGA data remain under their own terms.

---

**Dr. Anas Alzaeem**, Academic Director, London School of Multiomics & AI

<img src="https://raw.githubusercontent.com/MultiOmics-Intelligence/.github/main/assets/signature/LSMA_Banner_Anas_Alzaeem.png" alt="Dr. Anas Alzaeem, London School of Multiomics & AI" width="600">
