```markdown
# EukAlgae-T: Eukaryotic Algae Taxonomic Database and Analysis Pipeline

## Project Introduction
EukAlgae-T is the first specialized taxonomic database and analysis pipeline for accurate detection and ecological analysis of eukaryotic algae from whole metagenome shotgun sequencing data. This project provides a complete analysis pipeline including database alignment, filtering, annotation, and species statistics.

## Repository File Description

### 1. EukAlgae-T - Main Analysis Script
This is a Bash script containing the complete analysis pipeline:

**Main Functions:**
- Align FASTQ sequencing data to the eukaryotic algae database using Bowtie2
- Process alignment results with SAMtools, filtering out unaligned reads
- Apply MAPQ≥30 quality filtering and keep the best match for each read
- Process Contig IDs, removing interval parts to match database entries
- Add species information to alignment results using the annotation file (ALGAE_NAME_ID4.updated)
- Count the number of reads for each algal species

**Key Steps:**
1. Alignment: Align single-end or paired-end FASTQ files to the ALGAE.fna database using Bowtie2
2. Filtering: Retain alignments with MAPQ≥30 and select the best match for each read
3. Annotation: Associate alignment results with the ALGAE_NAME_ID4.updated file to add species information
4. Statistics: Generate a read count table for each algal species

### 2. Database Files
**ALGAE.fna - Eukaryotic Algae Reference Sequence Database**
- Format: FASTA format
- Content: Gene or genomic fragment sequences of eukaryotic algae
- Example record:
```
>CM008975.1:1217364-1224756
atggccacggcggctggccgctccagagagccgacgcccagcacgatgcgcacctggcac...
```
Sequence identifiers include genome accession numbers and position information

**ALGAE_NAME_ID4.updated - Species Annotation File**
- Format: Tab-separated text file
- Content: Maps sequence identifiers to species classification information
- Example record:
```
GCA_000002595.3 CM008962.1 Chlamydomonas reinhardtii 3055 3052 ...
```
- Column 1: Genome accession number
- Column 2: Sequence identifier (SOG GenBank query number with position information removed, corresponding to IDs in ALGAE.fna)
- Column 3: Species name (e.g., Chlamydomonas reinhardtii)
- Subsequent columns: NCBI taxid taxonomic identifiers

## Usage
### Dependencies
- Bowtie2: For sequence alignment
- SAMtools: For processing SAM/BAM files
- Bash: To run the script
- AWK: For text processing

### Running the Pipeline
1. Prepare data: Place all files in the same directory
 - EukAlgae-T (script file)
 - ALGAE.fna (database file)
 - ALGAE_NAME_ID4.updated (annotation file)
 - Paired-end sequencing files: name as *_1.fastq and *_2.fastq
 - Single-end sequencing files: name as *.fastq

2. **Build Bowtie2 index** (must be done before running the pipeline):
```bash
# Build index for the ALGAE.fna database
bowtie2-build ALGAE.fna ALGAE.fna

# This will create index files with prefix "ALGAE.fna":
# ALGAE.fna.1.bt2, ALGAE.fna.2.bt2, ALGAE.fna.3.bt2, ALGAE.fna.4.bt2
# ALGAE.fna.rev.1.bt2, ALGAE.fna.rev.2.bt2
```
3. Run the script:
```bash
# Grant execute permission
chmod +x EukAlgae-T

# Run the script
./EukAlgae-T
```

4. Output files:
 - For each input FASTQ file, the following will be generated:
   - *_reads{count}.txt: Processed alignment results, including read ID, Contig ID, species information, and taxonomic IDs
 - Final summary file:
   - algae_species_counts.tsv: Statistical table of algal species across all samples

### Usage Example
Assume the following file structure:
```
working_directory/
├── EukAlgae-T (script)
├── ALGAE.fna (database)
├── ALGAE_NAME_ID4.updated (annotation)
├── sample1_1.fastq
├── sample1_2.fastq
├── sample2.fastq
```

After running the script, the following will be generated:
- sample1_reads100000.txt (assuming 100,000 reads)
- sample2_reads50000.txt (assuming 50,000 reads)
- algae_species_counts.tsv: Summary of species statistics across all samples

## Output Description
### Intermediate File Format (Example):
```
ReadID    ContigID     SpeciesName              TaxonomicIDs
read1    CM008962.1    Chlamydomonas reinhardtii    3055 3052 ...
```

### Final Statistics File (algae_species_counts.tsv):
```
Algal Species          Reads Count
Chlamydomonas reinhardtii   1500
Thalassiosira pseudonana    800
...
```

## Customization and Extension
### Modifying Filter Thresholds
The script defaults to MAPQ≥30 as the filter threshold. Modify at:
```bash
awk 'BEGIN{OFS="\t"} $5 >= 30 {print $1, $3, $5}'  # Change 30 to other value
```

### Adjusting Thread Count
The script defaults to 8 threads for Bowtie2. Modify at:
```bash
bowtie2 -x "$INDEX" -1 "$f1" -2 "$f2" -S "$sam_file" -p 8  # Change 8 to other value
```

### Using Other Databases
If using other databases, ensure:
1. Database files are in FASTA format
2. Annotation file format is consistent with ALGAE_NAME_ID4.updated
3. Modify the INDEX variable in the script to point to the new database file

## Notes
1. Ensure Bowtie2 and SAMtools are correctly installed and in PATH
2. Adequate disk space is required for intermediate files
3. Paired-end sequencing files must follow the *_1.fastq and *_2.fastq naming convention
4. All files (script, database, annotation file, sequencing files) must be in the same directory
```
