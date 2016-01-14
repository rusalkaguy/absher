default: stats

SAMPLE ?= *
SAMPLE_PAT ?= merge*$(SAMPLE)*BQSR

stats: 
	ls $(SAMPLE_PAT)* | cut -d . -f 2- | sort | uniq -c| sort -k1,1nr

grid:
	@echo -e "sample|bai|bam.aa|bam.ab|bam.ac|bam.ad|bam|reorder.bam|reorder.bai" \
	; for x in $$(ls $(SAMPLE_PAT).bai); do \
		echo -en $$(basename $$x .bai)"|" \
		; for ext in bai bam.aa bam.ab bam.ac bam.ad bam reorder.bam reorder.bai; do \
			echo -ne $$(wc -c $$(basename $$x .bai).$$ext 2> /dev/null |awk '{print $$1}'); \
			echo -ne "|" \
		; done \
		; echo "" \
	; done \

bams:
	 make $(MAKE_FLAGS) $$(ls $(SAMPLE_PAT).bam.aa | perl -pe 's/\.aa//g;')
	 make $(MAKE_FLAGS) $$(ls $(SAMPLE_PAT).bam.aa | perl -pe 's/\.aa/.md5/g;')
	 make $(MAKE_FLAGS) $$(ls $(SAMPLE_PAT).bam.aa | perl -pe 's/\.aa/.flagstat.txt/g;')

qsub: 
	for x in $$(ls $(SAMPLE_PAT).bam.aa); do \
		echo qsub -N make_$$(basename $$x .aa); \
		qsub -N make_$$(basename $$x .aa) -l h_rt=23:00:00,s_rt=23:00:00,vf=1,h_vmem=1G -cwd -V -m beas -M $(USER)@uab.edu -j y -b yes \
			$$(which make) \
				$$(basename $$x .aa) \
				$$(basename $$x .aa).md5 \
				$$(basename $$x .aa).flagstat.txt \
				; \
	done
	nohup qstat_email_when_empty make_merge done in $(PWD) &

qsub_reorder: 
	for x in $$(ls $(SAMPLE_PAT).bam); do \
		echo qsub -N make_picard_reorder_$$(basename $$x .bam); \
		qsub -N make_picard_reorder_$$(basename $$x .bam) -l h_rt=23:00:00,s_rt=23:00:00,vf=31.8G,h_vmem=32G -cwd -V -m beas -M $(USER)@uab.edu -j y -b yes \
			$$(which make) \
				$$(basename $$x .bam).reorder.bam \
				$$(basename $$x .bam).reorder.bai \
				$$(basename $$x .bam).reorder.bam.flagstat.txt \
				; \
	done
	nohup qstat_email_when_empty make_pic done in $(PWD) &

reorder: 
	for x in $$(ls $(SAMPLE_PAT).bam.aa); do \
		echo picard_reorder_$$(basename $$x .aa); \
			$$(which make) $(MAKE_FLAGS) \
				$$(basename $$x .bam.aa).reorder.bam \
				$$(basename $$x .bam.aa).reorder.bai \
				$$(basename $$x .bam.aa).reorder.bam.flagstat.txt \
				; \
	done

.SECONDEXPANSION:
#%.bam: %.bam.aa %.bam.ab %.bam.ac $(wildcard %.bam.a?)
%.bam: %.bam.aa %.bam.ab %.bam.a?
	echo cat $^
	cat $^ > $@

%.bam.md5: %.bam
	md5sum $< > $@

LOAD_SAMTOOLS   ?= module load ngs-ccts/samtools/0.1.19

%.bam.flagstat.txt: %.bam
	$(LOAD_SAMTOOLS); samtools flagstat $< > $@

# correct chr order for GATK (M first)
PICARD_DIR 	?= /share/apps/ngs-ccts/picard-tools-1.79
GATK_REF_GENOME ?= /scratch/share/public_datasets/ngs/databases/gatk_bundle/2.8/hg19/ucsc.hg19.fasta

%.reorder.bam: %.bam $(GATK_REF_GENOME)
	PICARD_TEMP=$$(mktemp -d -p .) \
	; echo $$(date) start picard ReorderSam $* \
	; java -Xmx25g -Djava.io.tmpdir=$$PICARD_TEMP -jar $(PICARD_DIR)/ReorderSam.jar \
		I=$< \
		R=$(GATK_REF_GENOME) \
		O=$@ \
	; echo $$(date) DONE picard ReorderSam $* RC=$$? \
	; rm -rf $$PICARD_TEMP

%.reorder.bai: %.reorder.bam
	$(LOAD_SAMTOOLS); samtools index $< $@

download:
	cd ..; make mirror_$(PWD)


