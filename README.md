# Co-inoculation_v2
Scripts used on the co-inoculation v2 project

1. modify multifasta contig name

            awk '(FNR==1){f="B1_";sub(/.[A-Za-z]$/,"_",f)}/^>/{$0=">" f substr($0,2)}1' B1_finalpurged_primary.fasta > B1_finalpurged_primary_v2.fna
            awk '(FNR==1){f="DAOM197198_";sub(/.[A-Za-z]$/,"_",f)}/^>/{$0=">" f substr($0,2)}1' DAOM197198_Rhiir2.fna > DAOM197198_Rhiir2_v2.fna

2. Merge assemblies of DAOM and B1 to produce a reference template.

            cat DAOM197198_Rhiir2_v2.fna > Merged_DAOM197198_B1.fna
            cat B1_finalpurged_primary_v2.fna >> Merged_DAOM197198_B1.fna

3. Map reads with bbmap


        Sinteractive -c 16 -m 128G -t 02:00:00
        module load gcc bbmap/38.63
        bbmap.sh in1=C3-4_Rirregularis_1_val_1.fq in2=C3-4_Rirregularis_2_val_2.fq out=C3-4_mappedSuperZ17p.sam ref=../00_GenomeAssemblies/SuperAssembly_Z17ALL_primary_ctg.fasta
        
        # low quality repetitive assembly
        
        for i in $(ls Unmapped_Mesculenta_*.1.fq); do echo $i; bbmap.sh in1=$i in2=$(echo $i | cut -d'.' -f1).2.fq out=$(echo $i | cut -d'.' -f1)_mapped_MergedDAOM197198B1.sam ref=../../00_GenomeAssemblies/Merged_DAOM197198_B1.fna vslow k=8 maxindel=200 minratio=0.1 ; done


4. transform to bam sort and Qfilter
            
            module load gcc samtools
            samtools sort -m4G -@4 -o Z17Hifireads_RefSuperZ17p_sorted.bam Z17Hifireads_RefSuperZ17p.sam
            samtools view -bq 30 Z17Hifireads_RefSuperZ17p_sorted.bam > Z17Hifireads_RefSuperZ17p_sorted_Q30.bam
            samtools index Z17Hifireads_RefSuperZ17p_sorted.bam


5. FeatureCounts on chimer.

1. merge GTF file of both isolates.

            cat DAOM197198_Rhiir2_v2.gff > Merged_DAOM197198_B1.gff
            cat Rhizophagus_irregularis_B1.gff3 | grep -v "#" >> Merged_DAOM197198_B1.gff
            
2. run feature counts 

            /work/FAC/FBM/DEE/isanders/popgen_to_var/IM/ZZ_Soft/subread-2.0.3-source/bin/featureCounts -T 5 -a /work/FAC/FBM/DEE/isanders/popgen_to_var/IM/01_Coinoc_v2/00_GenomeAssemblies/Merged_DAOM197198_B1.gff -t exon -g ID -o Counts_COL2215.txt Unmapped_Mesculenta_COL2215_B1DAOM197198_1_mapped_MergedDAOM197198B1_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_B1DAOM197198_2_mapped_MergedDAOM197198B1_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_B1DAOM197198_3_mapped_MergedDAOM197198B1_Sorted_Q30.bam
          
            
            
            
            
            
            
            
            
            
            
            

5. Coverage analysis 

