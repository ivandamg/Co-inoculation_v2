# Co-inoculation_v2
Scripts used on the co-inoculation v2 project

# TEST merge GFF and fasta file into a chimera to map reads to the chimera

1. modify multifasta contig name

            awk '(FNR==1){f="B1_";sub(/.[A-Za-z]$/,"_",f)}/^>/{$0=">" f substr($0,2)}1' B1_finalpurged_primary.fasta > B1_finalpurged_primary_v2.fna
            awk '(FNR==1){f="DAOM197198_";sub(/.[A-Za-z]$/,"_",f)}/^>/{$0=">" f substr($0,2)}1' DAOM197198_Rhiir2.fna > DAOM197198_Rhiir2_v2.fna
            
Change header of genome file only chromosome code

            cat R_irregularis_DAOM197198_Kameoka.fna |sed 's/ Rhizophagus irregularis DAOM 181602=DAOM 197198 DNA, contig: unitig_*., whole genome shotgun sequence//' | sed 's/ Rhizophagus irregularis DAOM 181602=DAOM 197198 DNA, contig: unitig_*
.., whole genome shotgun sequence//' | sed 's/ Rhizophagus irregularis DAOM 181602=DAOM 197198 DNA, contig: unitig_*..., whole genome shotgun sequence//' > R_irregularis_DAOM197198_Kameoka_v2.fna

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
            
2. run feature counts on both mapped/reference B1 and DAOM independently

            /work/FAC/FBM/DEE/isanders/popgen_to_var/IM/ZZ_Soft/subread-2.0.3-source/bin/featureCounts -T 8 -a /work/FAC/FBM/DEE/isanders/popgen_to_var/IM/01_Coinoc_v2/03_MappedDAOM/DAOM197198_Rhiir2_v2.gff -t CDS -g ID -o Counts_REFDAOM_COL2215.txt  Unmapped_Mesculenta_COL2215_B1DAOM197198_3_mapped_DAOM197198_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_B1_1_mapped_DAOM197198_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_B1_2_mapped_DAOM197198_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_B1_3_mapped_DAOM197198_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_DAOM197198_1_mapped_DAOM197198_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_DAOM197198_2_mapped_DAOM197198_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_DAOM197198_3_mapped_DAOM197198_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_Mock_1_mapped_DAOM197198_Sorted_Q30.bam Unmapped_Mesculenta_COL2215_Mock_2_mapped_DAOM197198_Sorted_Q30.bam -p --countReadPairs
          


6. Call SNPs from RNA-seq BAM files

             # Call SNP
            module load Bioinformatics/Software/vital-it; module add UHTS/Analysis/freebayes/1.2.0; mkdir 03_SNP_Rhiir;
            for i in $(ls *.bam); do echo $i; freebayes --fasta-reference DAOM.fa --report-monomorphic -C 10 -p 1 $i > 03_SNP_Rhiir_$(echo $i | cut -d'_' -f1,2,3,4,5).vcf; done

            # Filter quality SNP Call

6. Orthologs between B1 and DAOM annotation.

Run orthologs analysis with orthofinder. then in excel identify wich are single-copy orthologs. 
Do normal DE analysis and then look if they are orthologs. always compare amf1 amf2 vs Co-inoculation


7. Then look DE genes with one reference. look at ortholog in other reference and check if they are also DE. If yes, it means that gene is activated in both isolates. If not it means that it is activated in a single-isolate.

8. Can confirm this by looking at SNP in the co-inoculation. We should expect presence of both variants if expressed in both isolates.
9. Coverage analysis 

            module load gcc bedtools2
            for i in $(ls *Q30.bam); do echo $i; bedtools genomecov -ibam $i -bga -split > $(echo $i | cut -d'_' -f3,4,5)_Covdetail.txt ; done


## TEST 2. Mapp reads individually to each genome assembly. and then with variant calling. identify to which isolate the counts are from. 

1. Map cassava unmapped reads to DAOM197198. 
2. Map cassava unmapped reads to B1.
3. Differential expression independently.
4. Variant calling to determine from wich isolate the counts come from: is the gene activated in one isolate or both? This can be done by comparing SNP.
5. chimera not a good aproach because a lot of genes will map together.

6. Review feature counts command to make it better.
