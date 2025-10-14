# Whole-Exome-Sequencing-Pipeline-SRR8879057

## **Overview**
In this pipeline, I created a whole exome sequencing pipeline for a familial case of congenital heart defect (SRA ID: SRR8879057), measured via validated variant calls by aligning reads with BWA, calling variants using GATK, and annotating them with VEP to give 45717 variants

The main objective of this analysis was to accurately identify high-confidence single nucleotide polymorphisms (SNPs) and small insertions/deletions (INDELs) across the human exome, and annotate their potential biological significance using the Ensembl VEP framework.

---

## **Methodology**

### **1. Data Preparation**
The paired-end FASTQ reads for the human sample **SRR8879057** were obtained from the [NCBI Sequence Read Archive (SRA)](https://www.ncbi.nlm.nih.gov/sra).  
After downloading, the files were transferred to the working directory for analysis.

### **2. Quality Control**
The quality of raw sequencing reads was assessed using **FastQC**, ensuring that read length distribution, base quality, and GC content met acceptable thresholds.  
This step helps identify potential issues like low-quality cycles or adapter contamination.

### **3. Read Trimming**
To improve read accuracy, **Trimmomatic** was used to remove adapters, low-quality bases, and short fragments.  
Paired and unpaired reads were generated, and the cleaned reads were re-checked with **FastQC** to confirm quality improvements.  
Some challenges, such as truncated file errors due to unstable internet connectivity, were resolved manually.

### **4. Read Alignment**
High-quality trimmed reads were aligned to the **human reference genome (GRCh38/hg38)** using **BWA-MEM**.  
This produced a SAM file containing aligned reads, which was then converted into BAM format using **Samtools** for efficient downstream processing.  
Sequence length mismatches between quality and alignment strings were fixed with an `awk`-based filtering approach to ensure consistency.

### **5. Sorting, Indexing, and Read Grouping**
The aligned BAM file was sorted and indexed using **Samtools**.  
Read groups were added with **GATK AddOrReplaceReadGroups**, ensuring compatibility for GATK’s downstream variant processing.

### **6. Duplicate Marking**
To eliminate redundant reads introduced during PCR amplification, **GATK MarkDuplicates** was applied.  
This minimizes bias in variant calling and ensures only unique reads contribute to variant discovery.

### **7. Base Quality Score Recalibration (BQSR)**
Using **GATK BaseRecalibrator**, systematic sequencing errors were corrected based on known variant sites such as dbSNP, 1000 Genomes, and Mills gold-standard indels (all in hg38 format).  
The recalibrated BAM file (`SRR8879057_recal.bam`) was generated to improve the reliability of downstream variant calls.

### **8. Variant Calling**
Variants were detected using **GATK HaplotypeCaller**, generating a raw VCF file containing SNPs and INDELs across the captured exome.

### **9. Variant Filtration and Merging**
SNPs and INDELs were separated and filtered using **GATK SelectVariants** and **VariantFiltration**, applying quality metrics such as:
- **QD (Quality by Depth)**  
- **QUAL (Confidence Score)**  
- **FS (Fisher Strand Bias)**  
- **MQ (Mapping Quality)**  

Filtered variants were then merged into a unified VCF using **GATK MergeVcfs**, and only **PASS** variants were retained via **bcftools**.

### **10. Functional Annotation (Ensembl VEP)**
The filtered variants were annotated using **Ensembl Variant Effect Predictor (VEP)**, which provides gene context, predicted impact, and functional consequence based on Ensembl’s GRCh38 cache.  
Annotation included data from multiple sources like **ClinVar**, **dbSNP**, **COSMIC**, **1000 Genomes**, and **gnomAD**.

---

## **Results**

### **VEP Summary**
- **Tool:** Ensembl VEP v113.0  
- **Reference Assembly:** GRCh38.p14  
- **Species:** *Homo sapiens*  
- **Execution Time:** 563 seconds (≈9.4 minutes)  
- **Input Variants:** 45,717  
- **Variants Processed:** 45,717 (0 filtered out)  
- **Novel vs Existing:** 819 novel (1.8%) / 44,898 existing (98.2%)  
- **Overlapped Genes:** 39,080  
- **Overlapped Transcripts:** 201,576  
- **Overlapped Regulatory Features:** 1,173  

---

### **Variant Classification**
| Variant Type | Count | Percentage |
|---------------|--------|------------|
| SNVs | 42,449 | 92.9% |
| Deletions | 1,787 | 3.9% |
| Insertions | 1,472 | 3.2% |
| Sequence Alterations | 9 | <0.1% |

---

### **Most Severe Consequences (Selected)**
| Consequence Type | Count |
|------------------|--------|
| Missense variant | 6,420 |
| Synonymous variant | 6,242 |
| Frameshift variant | 103 |
| Stop gained | 60 |
| Splice region variant | 1,072 |
| Intron variant | 18,548 |
| UTR variants (5'/3') | 2,274 combined |
| Non-coding exon variants | 8,129 |

**Most common effect:** *Intron and synonymous variants together account for over 60% of detected alterations.*

---

### **Coding Consequences**
| Category | Count |
|-----------|--------|
| Missense variants | 35,598 |
| Synonymous variants | 45,107 |
| Frameshift variants | 365 |
| Stop gained/lost | 275 combined |
| In-frame insertions/deletions | 582 |
| Other (retained/start codon, etc.) | <1% |

---

### **Functional Predictions**

#### **SIFT Summary**
| Prediction | Count | Percentage |
|-------------|--------|------------|
| Deleterious | 4,876 | 14.1% |
| Deleterious (low confidence) | 3,055 | 8.7% |
| Tolerated | 20,325 | 58.9% |
| Tolerated (low confidence) | 6,229 | 18.3% |

> Around **22.8%** of coding variants were predicted to be potentially deleterious.

#### **PolyPhen Summary**
| Prediction | Count | Percentage |
|-------------|--------|------------|
| Probably damaging | 2,415 | 7.0% |
| Possibly damaging | 2,417 | 7.0% |
| Benign | 27,425 | 80.5% |
| Unknown | 1,827 | 5.4% |

> The majority of variants were predicted to be **benign**, while about **14%** could be damaging.

---

### **Chromosomal Distribution**
The majority of variants were distributed across all autosomes, with the highest density observed on:
- **Chromosome 1 (4,535 variants)**  
- **Chromosome 11 (2,774 variants)**  
- **Chromosome 19 (2,606 variants)**  
- **Chromosome 2 (3,441 variants)**  

Sex chromosomes contained fewer variants:  
- **chrX:** 976  
- **chrY:** 75  

---

### **Positional Insights**
Variants were evenly distributed across protein-coding regions, with approximately:
- **8–9k variants per 10% interval** of protein length coverage,  
suggesting uniform detection across exonic regions.

---

### **Summary Interpretation**
The results indicate a successful end-to-end WES analysis with accurate alignment, variant calling, and high-quality annotation.  
The detected variants predominantly consist of **single nucleotide variants** (SNVs) with a large proportion of **synonymous and intronic changes**.  
Functional annotations show that most variants are likely **benign or tolerated**, though a meaningful subset exhibits **deleterious potential**, warranting deeper clinical or population-level investigation.

---

## **Environment Requirements**

- **Operating System:** Linux / macOS (Apple Silicon compatible)
- **Reference Genome:** GRCh38.p14 (hg38)
- **Tools:**
  - FastQC  
  - Trimmomatic  
  - BWA  
  - Samtools  
  - GATK 4.6.2.0  
  - bcftools  
  - Ensembl VEP  

---

## **Reproducing the Workflow**

1. **Clone the Repository**
   ```bash
   git clone https://github.com/<your-username>/WES_Pipeline_SRR8879057.git
   cd WES_Pipeline_SRR8879057
