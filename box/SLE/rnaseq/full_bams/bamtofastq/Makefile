LOAD_BEDTOOLS = module load ngs-ccts/bedtools/2.23
LOAD_SAMTOOLS = module load ngs-ccts/samtools/0.1.19

SAMPLE_LIST ?= SL62331
FASTQ_LIST ?= $(foreach READ,R1 R2,$(foreach SAMPLE_ID,$(SAMPLE_LIST),mergeLanes_$(SAMPLE_ID)_rmDup_realign_BQSR.$(READ).fastq))
default: $(FASTQ_LIST)

%.R1.fastq %.R2.fastq: ../%.bam 
	$(LOAD_BEDTOOLS); $(LOAD_SAMTOOLS); \
	samtools sort -n $< - | bedtools bamtofastq -i - -fq $*.R1.fastq -fq2 $*.R2.fastq 
