The following steps will guide you through the TPS pipeline:


STEP0 - Converting genotype file
-----------------------------------
In this step, you need to obtain you genotype file in plink format (with ped\map or bed\bim\fam). 
If the files are in EIGENSTRAT format, use the CONVERTF tool to convert to plink format:

for that, create a file (e.g., par.EIGENSTRAT.PED.aDNA) with the following lines:

	genotypename:    filename.geno
	snpname:         filename.snp
	indivname:       filename.ind
	outputformat:    PED
	genotypeoutname: filename.ped 
	snpoutname:      filename.map 
	indivoutname:    filename.ind

and run:
/home/EIG-master/bin/convertf -p /home/EIG-master/CONVERTF/par.EIGENSTRAT.PED.sDNA

Next, retain only the SNPs in Step 1\SNP_list.snp.

Run: 
	plink --bfile DataS1_all_SNPs --extract SNP_list.snp --make-bed --out DataS1

Obtain the list of SNPs that you used:
	cut -f2 DataS1.bim > SNP_list.txt

STEP1 - Merging your file with the temporal components
-------------------------------------------------------
In this step, you will merge your plink formatted genotype file (.map, me.ped) with the temporal components. 
First, trim the temporal components to the number of SNPs that are left in your database.

	mv TemporalComponents_8_GP.* TemporalComponents_8_GP_temp.*
	plink --bfile TemporalComponents_8_GP_temp extract SNP_list.txt --make-bed --out TemporalComponents_8_GP
	
Now, merge the files	

	plink --bfile TemporalComponents_8_GP --merge DataS1.bed DataS1.bim DataS1.fam --make-bed --out merge
	
plink will create files merge.bed, merge.bim and merge.fam that you can use in the following step.
Update the merge.pop manually by adding spaces with the same number as the sample number.

STEP2 - Calculating the temporal components of your samples
-----------------------------------------------------------
Run admixture on the merged bed file. Don't forget to put the merge.pop file in the same folder

	admixture merge.bed -F 8 -j12
	
This will produce output files merge.8.P and merge.8.Q. 
The top lines of the merge.8.Q (just the first rows corresponding to the test samples) are used as input to the TPS algorithm.

STEP3 - Predicting 'merge.8.Q' data file
---------------------------------------------------------
In this step, TPS is used (TPS(running).ipynb) to predict merge.8.Q, the output of ADMIXTURE. Only the first 961 rows are of interest and are the input for TPS.
To run TPS(running).ipynb, merge.8.Q or your desired input and the model (random_model.pkl) must be in the same directory as they are now. 
All you have to do is run the file with your dataset or the default dataset (merge.8.Q).