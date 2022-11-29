# Extraction-Pipeline-CJ-Payout-

From Savings United we received files through file zilla using the following credentials: 

SFTP CREDENTIALS: 




  Hostname: 34.88.209.39
  Username: cj
  Folder: /files/incoming


1) 	Monthly Load files from FileZilla-SFTP to Google Drive (we split files manually into the corresponding subfolder depends on the publisher_id). 
 Drive Path:   https://drive.google.com/drive/u/0/folders/1MIlXPDG09Df6QpKCVfK8JvyD1q_sVrGX

2)	Run Python script: “run_payout_dev.py” loading files from Drive to SQL tables. Before load data, the script truncates tables to avoid duplicating values.

	Important for BI: 
	BEFORE run the python script we need to check in sql management studio (job activity monitor) the jobs: dwh_etl_an_payout & dwh_etl_an_last_3:  THEY MUST NOT BE RUNNING. Those files already exist in the history table, not in the raw (raw table is being truncated every time the python script runs) Reason: We always have the files in google drive.  
    

3)	Move FileZilla files from SFTP incoming folder to SFTP processed folder.


4)	Load Data from drive to SQL tables:  

Step 13 from job: dwh_etl_an_payout Executes Stored procedure: 
 Updating_an_commission_junction_payout  
 
 We should be careful and not start this job from step 1. It must start from step 13. For CJ we run it manually after we process the files. → To run it manually we execute this sp manually using “exec” function in sql query, because if we run job from step 13, may run step 14 too and that is not necessary.


In that Stored Procedure we transform data:
Update publisher/site currency of the data being loaded 
Non-zero transaction ids are processed according regular rules: Delete if exist any an_id = 0		 	
		      -	Update subid_site_id and subid_shop_id 
                             -     Adspace site id, shop id & subid site id, shop id
                             -     Update adspace_site_id, subid_site_id using website_id 
		     -     Delete if any duplicated raw exist


5)	Looker load 
REBUILD REPORTS LOOKER: 
STEP 15 ((SUNDAYS 4 AM )):
How we execute it? 
Manually from step 15 
Or running in sql query: 
exec msdb.dbo.sp_start_job N'dwh_post_rebuild_payout_reports';
