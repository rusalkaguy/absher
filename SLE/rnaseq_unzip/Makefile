SHELL := /bin/bash
# source data
GENES_TABLE ?= isoform2gene_v24mapped.txt
#GENE_TABLE ?= gene_table_format2.txt
SAMPLE_METADATA ?= ../rnaseq/samples.rch2.txt
SAMPLE_LIST = $(basename $(basename $(wildcard SL*.isoforms.fpkm_tracking)))

# derrived data
SAMPLE_2_PHENO_SUBSET ?= sample_2_pheno_subset.txt
PHENO_SUBSET_LIST = $(addsuffix .gene_isosum.pheno_subset.txt, $(SAMPLE_LIST))
# region/sample of interest
GENE_NAME  ?= TNFRSF4
GENE_TABLE ?= $(GENE).$(GENES_TABLE)
SAMPLE	?= SL62330
SAMPLE_ISO	?= $(SAMPLE).isoforms.fpkm_tracking
SAMPLE_GENE	?= $(SAMPLE).genes.fpkm_tracking

default: $(PHENO_SUBSET_LIST)

# utils
unzip_all:
	for x in $$(ls -1 ../rnaseq/*.gz); do \
		D=$$(basename $$x .gz); \
		if [ ! -e "$$D" ]; then \
			echo gunzip $$x; \
			gunzip -c  $$x > $$D; \
		fi; \
	done

# 
# merge isoforms into correct genes sums
#
%.gene_isosum.txt: %.isoforms.fpkm_tracking $(GENES_TABLE)
	echo -e "#sample_id\tgene_name\tgene_id\tfpkm" > $@
	awk -v COLUMN=0 -v DEBUG=0 -v GENE_NAMES=1 -v SAMPLE_NAME=$* -v GENE_TXN_MAP=$(GENES_TABLE) \
	-f ~/uab_ngs/rnaseq/cufflinks/sum_fpkm.awk $< \
	>> $@

$(SAMPLE_2_PHENO_SUBSET): $(SAMPLE_METADATA)
	awk 'BEGIN{FS="\t";OFS="\t"}/^#/{printf "#"}{key=$$4"|"$$5;print $$3,key,$$4,$$5}' $< \
	| sort -k1,1 \
	> $@

%.gene_isosum.pheno_subset.txt: %.gene_isosum.txt $(SAMPLE_2_PHENO_SUBSET)
	join -t "	" $^ | \
	awk 'BEGIN{FS="\t";OFS="\t"}/^#/{printf "#"}{key=$$2"|"$$3;mkey=key"|"$$5;print $$1,$$2,$$3,$$4,mkey,key,$$5,$$6,$$7}' \
	> $@


# merge
all.gene_isosum.pheno_subset.txt: $(PHENO_SUBSET_LIST)
	cat $^ > $@

all.gene_isosum.pheno_subset.thin.txt: all.gene_isosum.pheno_subset.txt
	awk 'BEGIN{FS="\t";OFS="\t"}/^#/{printf "#"}{print $$5,$$4}' $< > $@


# IAP/CD47
ACCESSION ?= ENSG00000196776
subsets_with_gene:
	join -1 1 -2 1 \
		<(grep $(ACCESSION) *genes* | awk '{split($$1,a,"."); print a[1],$$5,$$11;}' | sort | uniq) \
		<(cut -f 3,4,5 ../rnaseq/samples.rch2.txt| sort -k1,1) \
	| sort -k5,5 -k4,4 \
	| awk 'BEGIN{OFS="\t"}{print "$(ACCESSION)",$$0}'


# debug
debug_1: 
	echo "egrep TNFRSF4"
	egrep -w "(ENST00000379236|ENSG00000186827|ENST00000453580)" SL62330.isoforms.fpkm_tracking 

debug:  $(SAMPLE_ISO) $(GENES_TABLE)
	echo "sum_txns2 all"
	awk -v COLUMN=0 -v DEBUG=0 -v GENE_NAMES=1 -v SAMPLE_NAME=$(SAMPLE) -v GENE_TXN_MAP=$(GENES_TABLE)  -f ~/uab_ngs/rnaseq/cufflinks/sum_fpkm.awk $<

debugg:  $(SAMPLE_GENE) $(GENE_TABLE)
	echo "sum_txns2 all"
	awk -v COLUMN=0 -v DEBUG=0 -v GENE_NAMES=1 -v SAMPLE_NAME=$(SAMPLE) -v GENE_TXN_MAP=$(GENES_TABLE)  -f ~/uab_ngs/rnaseq/cufflinks/sum_fpkm.awk $<


$(GENE_TABLE): $(GENES_TABLE)
	grep "^#"  $< > $@
	grep -v "^#" $<  | egrep -w "$(GENE)" $< >> $@

debug_gene: $(SAMPLE_ISO) $(GENE_TABLE) 
	echo "sum_txns2 $(GENE)"
	awk -v DEBUG=1 -v GENE_NAMES=1 -v SAMPLE_NAME=SL62330 -v GENE_TXN_MAP=$(GENE_TABLE)  -f ~/uab_ngs/rnaseq/cufflinks/sum_fpkm.awk $<

test: 
	@echo SAMPLE_LIST=$(SAMPLE_LIST)
	@echo PHENO_SUBSET_LIST=$(PHENO_SUBSET_LIST)

