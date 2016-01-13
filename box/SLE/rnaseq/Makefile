default: download

download: fpkm_tracking bams 

%: mirror_%.lftp
	lftp -f $<

qsub_%: mirror_%.lftp
	qsub -N lftp_$* -l h_rt=23:00:00,s_rt=23:00:00,vf=1,h_vmem=1G -cwd -V -m beas -M $(USER)@uab.edu -j y -b yes $$(which lftp) -f $<
	nohup qstat_email_when_empty lftp lftp_bams &



