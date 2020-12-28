# NICU_Microbiome_Study
Microbiome analysis and associated data.

This repo contains supplementary data for the manuscript titled "Characterising the bacterial gut microbiome of very preterm infants". 
This includes: 

 - The full workflow from raw reads to analysis. 
 - The metadata (deidentified).

## Summary of methods
### Lab protocols
 - DNA extraction was conducted using the Bioline ISOLATE Fecal DNA Kit, with modifications made in consultation with the manufacturer to optimise DNA yield. This included increased beta-mercaptoethanol (from 0.5 to 1% to increase DNA solubility and reduce secondary structure formation), addition of an extra wash step (to improve purity) and decreased elution buffer volume (to increase final DNA concentration). 
 - For library preparation we followed the Illumina metagenomics library preparation protocol, using the Index Kit v2 C, along with Platinum™ SuperFi™ PCR Master Mix. 
 - For sequencing, we used the MiSeq Reagent Kit V3 was used in combination with the Illumina MiSeq System, targeting the V3 and V4 regions with the 785F/800R primer combination.

### Bioinformatics
 - Pre-analytical bioinformatics were conducted in R Studio Version 3.6.1 (63). 
 - A pipeline was adapted from [Workflow for Microbiome Data Analysis: from raw reads to community analyses](https://bioconductor.org/help/course-materials/2017/BioC2017/Day1/Workshops/Microbiome/MicrobiomeWorkflowII.html#abstract).
 - [*DADA2*](https://pubmed.ncbi.nlm.nih.gov/27508062/) was used for quality filtering and trimming, demultiplexing, denoising and taxonomic assignment (with the SILVA Database), and the microDecon package used to remove homogenous contamination from samples using six blanks originating in extraction.
 - More detail on the bioinformatics can be found in the workflow file (.pdf and .rmd).

### Statistical analyses

#### Exploring changes in composition and diversity from admission to discharge
 - For statistical analysis, a [*phyloseq*](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0061217) object was created. The package allows storage, analysis and plot creation of complex phylogenentic sequence data associated with taxonomy, metadata and taxonomic trees.
 - To construct the taxonomic tree, we began with multiple alignment using the *DECIPHER* package, then tree construction with the *Phangorn* package. 
 - This resulting phylogenetic tree, previously created ASV and taxonomy tables (from the bioinformatics workflow), and a metadata file were combined to create a phyloseq object that would be used for downstream analysis.
 - Taxa were then filtered by prevalence (threshold 0.01), agglomerated at the genus level and then normalized through Total Sum Scaling (TSS). 
 - TSS was used as the log-transformation inherent in other normalization methods can over-estimate low abundant taxa, skewing the results. 
 - The data were then explored through Principle Coordinate Analysis (PCoA) plots using a Bray-Curtis dissimilarity matrix. 
 - PERMANOVA was then conducted for community-level comparisons between Admission and Discharge samples to observe group-level differences based on the Bray-Curtis dissimilarity matrix, using the `adnois()` function of the package *Vegan*. This fits linear models to distance matrices and uses a permutation test with pseudo-F ratios to calculate variance. Homogeneity of group variances was calculated using betadisper(), which utilizes the PERMDISP2 procedure. 
 - Alpha diversity indices, Shannon Index and Observed (richness), were calculated on filtered, non-agglomerated data using phyloseq. Richness is a measure of the number of ASVs present, where as the Shannon diversity builds on this by including eveness in the caclulation of its index.
 - A comparison for alpha divresity was made between Admission and Discharge samples using a Wilcoxon Rank Sum Test, with adjusted p-values (Benjamini-Hochberg procedure).
 - To compare differential abundance between admission and discharge samples, data that was filtered and agglomerated at the genus level, but not transformed, were then normalized and modeled (negative-binomial) with *DESeq2*. Rather than working with log-transformed data, DESeq2 considers a variance stabilizing transformation that fits a Negative Binomial generalized linear model (per a row of amplicon sequence variants) while taking into account a normalizing offset for sequencing depth.
 - Then with the `DESeq()` function, a Wald Test (with Benjamini-Hochberg multiple inference correction) was performed to determine significant differentially abundant taxa.

#### Exploring the effect of clinical variables on alpha diversity and taxonomic abundance
Multivariant models were then used to explore the impact of clinical variables on microbial populations, specifically Shannon Diversity and taxonomic abundance.

##### Shannon Diversity
- For Shannon diversity, a mixed effects linear regression model was created using the package [*lme4*](https://cran.r-project.org/web/packages/lme4/vignettes/lmer.pdf), with a gaussian distribution and using the restricted maximum likelihood  estimation. 
- Continuous predictors were scaled and centered to avoid convergence issues and multicollinearity assessed using the *AED* package. 
- Gestation and birth weight were collinear and so birth weight was removed from the model. 
- Thirteen predictors: mode of delivery, feeding type, gestation, antenatal antibiotics, antenatal infections, NEC, sepsis, chorioamnionitis, neonatal antibiotics, death, prolonged membrane rupture, preeclampsia, diabetes and retinopathy of prematurity were included in the initial model. 
- To control for high inter-individual variation, individual’s identification  (URN shown as ID in the metadata attached - deidentified) was included as a random factor. 
- To assess the influence of clinical variables at both admission and discharge, the type of sample was included as an interaction variable. 
- Resulting model = Shannon ~ (13 Parameters) * Type + (1|URN).
- Backwards selection was then implemented. This is a method to simplify the model by comparing Akaike’s Information Criterion (AIC) scores between regression models and removing predictors that are not contributing to the model. 
- The process was repeated until the least complex adequate model was identified. This is when no more predictors could be removed without significant effects. 
- Final model = Shannon ~ (Sepsis + Feeding Type + Chorioamnionitis + (Mode of Delivery + Gestation Days + NEC + Preeclampsia + ROP)) * Type + (1|URN). 
- The significance of the fixed effects variables in this final model was then assessed using analysis of deviance (Type II Wald Chi-square test) from the *car* package, and post-hoc pairwise Tukey comparisons (correcting for multiple comparisons) from the *emmeans* package. 

#### Taxonomic abundance
 - For differential taxonomic abundance, two negative binomial generalized linear models were created using the package *DESeq2*. 
 - A combination of previous literature and exploratory analysis (PCoA plots, PCA and scatterplots) were used for model selection. 
 - Continuous predictors were scaled and centered, and multicollinearity assessed. 
 - Taxa were agglomerated at the genus level, due to the limited sequencing depth of short amplicon sequencing. 
 - To reduce the number of false positives, two separate models were run. One for admission samples and another for discharge samples (I found the more complicated the model, the more "significant" variables I would get). 
 - Model = Taxonomic Abundance ~ Sepsis + Feeding Type + Chorioamnionitis + Mode of Delivery + Gestation Days + NEC + Preeclampsia + ROP.
 - Low abundance and low frequency taxa were then removed, and a Wald Test with the Benjamin-Hochberg multiple inference correction performed.

**The full workflow and more are found in this repo**
