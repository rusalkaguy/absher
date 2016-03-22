LOAD_SAMTOOLS	= module load ngs-ccts/samtools/0.1.19
LOAD_ZLIB	= module load ngs-ccts/zlib/1.2.8
LOAD_TOPHAT 	= $(LOAD_ZLIB); module load ngs-ccts/tophat/2.0.13
NSLOTS		?= 1
NRAM		?= 5
SAMPLE_LIST ?= SL62331
FASTQ_LIST ?= $(foreach READ,R1 R2,$(foreach SAMPLE_ID,$(SAMPLE_LIST),mergeLanes_$(SAMPLE_ID)_rmDup_realign_BQSR.$(READ).fastq))
default: $(FASTQ_LIST)

# sorted bam
# -n sort by name
# -@ threads
# -m max memory PER THREAD; [786M]
../%.sbam: ../%.bam
	$(LOAD_SAMTOOLS); samtools sort -n -@ $(NSLOTS) -m $(NRAM) $< $@

# -Q = include reads failing quality control
%.R1.fastq %.R2.fastq: ../%.sbam
	$(LOAD_TOPHAT); \
	bam2fastx --fastq -Q --all -o $@ --paired -N $<

info:
	echo SAMPLE_LIST = $(SAMPLE_LIST)
	echo FASTQ_LIST = $(FASTQ_LIST)
	echo NSLOTS= $(NSLOTS); NRAM=$(NRAM)


