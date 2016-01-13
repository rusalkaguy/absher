default:
	ls merge* | cut -d . -f 2- | sort | uniq -c| sort -k1,1nr

bams:
	 make $(MAKE_FLAGS) $$(ls *.aa | perl -pe 's/\.aa//g;')
	 make $(MAKE_FLAGS) $$(ls *.aa | perl -pe 's/\.aa/.md5/g;')
	 make $(MAKE_FLAGS) $$(ls *.aa | perl -pe 's/\.aa/.flagstat.txt/g;')

qsub: 
	for x in $$(ls *.aa); do \
		echo qsub -N make_$$(basename $$x .aa); \
		qsub -N make_$$(basename $$x .aa) -l h_rt=23:00:00,s_rt=23:00:00,vf=1,h_vmem=1G -cwd -V -m beas -M $(USER)@uab.edu -j y -b yes \
			$$(which make) $$(basename $$x .aa)  $$(basename $$x .aa).md5  $$(basename $$x .aa).flagstat.txt; \
	done
	nohup qstat_email_when_empty make_merge done in $(PWD) &

.SECONDEXPANSION:
#%.bam: %.bam.aa %.bam.ab %.bam.ac $(wildcard %.bam.a?)
%.bam: %.bam.aa %.bam.ab %.bam.ac %.bam.a?
	echo cat $^
	cat $^ > $@

%.bam.md5: %.bam
	md5sum $< > $@

LOAD_SAMTOOLS   ?= module load ngs-ccts/samtools/1.2

%.bam.flagstat.txt: %.bam
	$(LOAD_SAMTOOLS); samtools flagstat $< > $@

download:
	cd ..; make mirror_$(PWD)


