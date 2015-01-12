Project:  Trucking Division Modeling
Purpose: Describe and document the data build process for Trucking project.
Contact(s): Paul Detchemendy, Jenna Ahn, Xiaomiao Tang, Jae Han
Date: September 2012
________________________________________________________________________


I.  Data Description
This project uses data compiled by the Federal Motor Carrier Safety Administration (FMCSA).  The FMCSA maintains a database on motor carriers called the Motor Carrier Management Information System (MCMIS), which includes inspection, crash, compliance, safety audit, and registration information. Upon our request, the monthly data is transferred to us via FTP every month by Computing Technologies, Inc (COTS), which is the company that handles the data distribution for FMCSA. This data is also available on the FMCSA website http://safer.fmcsa.dot.gov/.  
L&I data information should be included here.
 

II.  Data Acquisition
Around the 6th day of each month, an MCMIS order form is e-mailed (currently by Kristine Palomares) to Francisco Guzman, our contact at COTS. Our request specifically asks for Census, Inspection, and Crash raw data files for the current year and the prior two full years. Until August of 2012, we received a data CD around 2 weeks following the order. However, the data is now transferred to us via FTP. An analyst copies the contents of the 3 zipped folders in the FTP onto his/her local drive and then unzips the contents into the same folder. 
L&I data information should be included here.  The approach is similar; we will be receiving the data via an FTP server every month.
	

III.  DataBuild Process Documentation 
This section describes the programs that are run in the shell program. Scores are calculated for the most recent period since the last data transfer. These SAS program files and a process flow diagram of this process can be found in the following locations: 
	
	SAS program files: insert filepath for final location of program files
	Process flow diagram: 
S:\Global Casualty\Trucking\Documentation\Data Descriptions\Trucking_DataBuild_FlowDiagram_v01.vsd

A.	Import raw data files

p100_census 

Overview:
In this program, the raw text file for the FMSCA Census data (CensusPubAsOfYYYYMMDD.txt) is read and converted to a SAS dataset called d100_census. The Census data provides the most recent snapshot of the motor carriers’ company information. The Census raw text file comes with documentation (Word file) that lists and explains all the fields in the raw text file.

Additional Notes:
•	The field names and their order in the text file should not change with each monthly update. However, this may happen from time to time, so the log should be checked to confirm consistency. 		

p110_crash
Overview:
In this program, the 2 raw text files for the FMSCA Crash data (CrashMasterMMDDYYYY_MMDDYYYY.txt and CrashCarrierMMDDYYYY_MMDDYYYY.txt) are read and converted to 2 SAS datasets: d110_crashmaster and d110_crashcarrier. The d110_crashmaster table contains information (e.g. date, location, injuries, etc) about each crash event that was reported during the period indicated in the filename. Each crash event is identified by a unique CRASH_ID. The d110_crashcarrier table contains information about the motor carriers (identified by CRASH_CARRIER_ID) involved in the crash events. The Crash raw text files come with documentation (Word file) that lists and explains all the fields in the raw text files.

Additional Notes:
•	The field names and their order in the text file should not change with each monthly update. However, this may happen from time to time, so the log should be checked to confirm consistency.
•	The macro tbx_getdups is used on both output tables of this program to check if there are any duplicate records. (Once dups are detected, further investigation will be conducted to find the causes of the dups and, we will use the “nodupkey” or “nodup” function to remove them if necessary. 

p120_inspection

Overview:
In this program, the 3 types of raw text files (there are 3 each for the prior 3 years for a total of 9) for the FMSCA Crash data (Insp_Pub_MMDDYYY_ MMDDYYY.txt, Insp_Unit_Pub_ MMDDYYY _ MMDDYYY.txt, and Insp_Viol_Pub_ MMDDYYY _ MMDDYYY.txt) are read and converted to 3 SAS datasets: d120_inspection, d120_inspection_uni, and d120_inspection_vio. 

The d120_inspection dataset contains information (e.g. date, location, injuries, inspection type, violations, etc) about each inspection event that was reported over the period indicated in the filename. Each inspection event is identified by a unique INSPECTION_ID. The d120_inspection_uni dataset is a reference table that contains the INSP_UNIT_ID identifier associated with each inspection event. The d120_inspection_vio dataset is a reference table that contains the INSP_VIOLATION_IDs associated with each inspection that resulted in a violation. The Inspection raw files come with documentation (Word file) that lists and explains all the fields in the raw text files. 

Additional Notes:
•	The field names and their order in the text file should not change with each monthly update. However, this may happen from time to time, so the log should be checked to confirm consistency.
•	The macro tbx_getdups is used on the output tables of this program to check if there are any duplicate records. (Once dups are detected, further investigation will be conducted to find the causes of the dups and, we will use the “nodupkey” or “nodup” function to remove them if necessary. 



B.	Perform cleanup on newly created SAS datasets

p200_census

Overview:
In this program, variables that are not needed downstream are dropped from the d100_census dataset, the CENSUS_NUM field is renamed to DOT_NUMBER, and a temporary d200_census_cur dataset is created which represents the current month’s snapshot. The d200_census_cur dataset is then concatenated together with the d200_censusmonthly dataset (which contains Census data for the prior 3 years) to create the d200_census dataset.

Additional Notes:
•	The most recent version of the d200_censusmonthly needs to be placed in the appropriate folder before the program is run. This will be the d200_census dataset from the prior month’s update. Rename this dataset to d200_censusmonthly and place in the appropriate folder before running the program.


p220_inspection

Overview:
In this program, the newly created inspection datasets are cleaned and merged. The d120_inspection_uni and d120_inspection_vio datasets are merged to form the d220_inspection_univio dataset. Units with buses are removed from this dataset since we are not concerned with carriers with buses. The violations for each unit are then summed and maxed and outputted to the d220_inspection_univio2_summax dataset.

Additional Notes:
•	The sum of the violations is determined for each inspection, to give the count of the violations for a given inspection.
•	The max of the violations is determined for each inspection, which indicates there was at least one instance of a certain violation type for that inspection.


p310_crash

Overview:
In this program, the newly created d110_crashmaster dataset is cleaned. Any crashes involving buses or missing DOT_NUMBER information are removed. Afterwards, in a series of several steps, duplicate records are removed from d110_crashmaster. 

Additional Notes:
•	This step is meant to remove junk records out of the crash events table, which is very important for modeling purposes.


C.	Merge the MCMIS datasets together

p400_census

Overview:
In this program, the d400_census dataset is created from merging the d200_census, d110_crashcarrier, d310_crashmaster, d120_inspection, and d220_ins_univio2_summax datasets. The d200_census dataset is ready for merging without any further steps. The d110_crashcarrier and d310¬_crashmaster datasets are merged by the CRASH_ID and rolled up to the level of DOT_NUMBER and PERIOD. Next, the d120_inspection, and d220_ins_univio2_summax datasets are merged by the INSPECTION_ID and rolled up to the level of DOT_NUMBER and PERIOD. The last step is the merging of the Census, Crash, and Inspection datasets.  

Additional Notes:
•	The variable Vio_Count is created in this program. This variables counts all the violations based on the entries in the various violation fields. We create this variable due to the discrepancy between the total count of the entries in the violation fields and the VIOL_TOTAL field in the original raw file.


D.	Construct model variables

p500_vars

Overview:
In this program, modeling variables are created and outputted to the d500 dataset. These variables are ObsSinceLastInspec, ObsSinceLastCrash, DaysSinceLastActivDate, Inact_Insp, Inact_Crash, Inact_Dates, Inact_2yr, YrsSince_MCS150, and DataAsOf_MCMIS.   

Additional Notes:
•	ISO Regions for each trucking company are assigned using the Region_ISO format names found in the Data_for_SAS_TRK1.xls spreadsheet and the trk_Formats_XT macro. This spreadsheet and macro program need to be in the trk folder while running the program. 
•	The POP1 variable created in this program flags the target universe of trucking companies. The flagged companies are located in the U.S., have a positive number trucks, no buses, and an ICCDocket of MC.


E.	Calculate risks scores

p550_score

Overview:
In this program, annualized input variables for the predictive model are created, and risk scores (1 to 4) are calculated using predictive model coefficients. Before the scores are calculated, flags are set for the various reasons why a model cannot be scored. These reasons include fleet size being greater than 50, total mileage information being missing, fleets having buses or no trucks, etc. The truck carriers that are not flagged for being non-scorable are scored by inputting 8 variables (On_Insp, On_Insp_Vio, flg_crash, Vio_Driver_SPEDNG, BrkAny_Flg, DRIVER_VIOL_TOTAL, count_crash, UpdatedTrucksCount/tot_trucks) into the predictive model.

Additional Notes:
•	The thresholds for the risk ranks (1 to 4) are written into the %risk_rank macro, which is stored in the p550_riskrank program. The thresholds are specified for each fleet size from 1 to 20, and in increments of five from 21 to 50.  


F.	Add Insurance data

p570_insurance

Overview:
In this program, the insurance master dataset, Ins0611master2, is merged with the p550_score dataset by DOT_NUMBER and PERIOD to create the d570 dataset. (In new version of master dataset, we are working on bringing in both Insurance and License data in order to filter out companies not in business during certain periods)


G.	Finalize dataset for website update

p600_finalizedataset

Overview:
In this program, the d570 dataset is cleaned one more time (i.e. remove a variable and change a label) to generate the d600_DOT_Master dataset. The d600_DOTM_CurrMo_Website dataset used to update the website is created by extracting the most recent month’s data out of the d600_DOT_Master dataset.      

Additional Notes:
•	Before running this program, the CurrPeriodForWebSiteFile parameter needs to be manually updated to the proper month in YYYYMM format. This parameter defines the most recent period’s data that will be used to create the d600_DOTM_CurrMo_Website dataset.
•	A dataset that shows the history of ranks for each DOT_NUMBER is created at the end as a reference. This dataset is called d600_hist_pfm.






 
Appendix A¬ – List of SAS Programs


SAS Program	Description	Input Dataset(s)	Output Dataset(s)
p100_census	Import raw data	Raw CENSUS file	d100_census
p110_crash	Import raw data	Raw CRASH files	d110_crashcarrier, d110_crashmaster
p120_inspection	Import raw data	Raw INSPECTION files	d120_inspection, d120_inspection_uni,
d120_inspection_vio
p200_census	Clean/Validate data	d100_census	d200_census
p220_inspection	Clean/Validate data	d120_inspection, d120_inspection_uni,
d120_inspection_vio	d220_ins_univio2_flag, d220_ins_univio2_summax

p310_crash	Clean/Validate data	d110_crashmaster	d310_crashmaster
p400_combine	Merge datasets	d200_census, d310_crashmaster,
d110_crashcarrier,
d120_inspection,
d220_ins_univio2_summax	d400_base
p500_vars	Create model variables	d400_base	d500
p550_score	Calculate risk scores	d500	d550
p570_insurance	Add insurance data	d550, Ins0611master2	d570
p600_finalizedatasets	Prepare web datasets	d570	d600_DOT_Master


