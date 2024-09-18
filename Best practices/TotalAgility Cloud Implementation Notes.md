# Purpose
This document aims to provide some guidelines for Kofax KTA Cloud implementations.
At the time this document was written, Kofax Professional Services have completed several KTA Cloud implementations. There have been several issues and observations from the KTA Cloud implementations thus far. This document aims to capture some of the guidelines for Kofax resources who are working with KTA Cloud to overcome/avoid those issues in future implementations.

# Data Retention

```
Note: (Update Sep/2024) From TA version 8.0 some retention settings are managed by TA CloudOps.
Professional Services shall submit the retention periods required via a TS case.
Use the spreadsheet provided to specify the retention periods required.
```

Retention policies are important in any implementation, whether it's on-premises, private cloud, or public cloud. However, when it comes to a SaaS (Software as a Service) environment it is even more important to apply the appropriate retention polices and configurations to avoid large amounts of data accumulating in the SaaS infrastructure.
Exponential data growth results in additional costs, maintenance overheads and performance issues and ultimately customer dissatisfaction.
Therefore, we should encourage our customers to apply shorter retention periods. Highlight to customers that their KTA platform is not to be treated as a system of record and it is not meant to hold transaction/job data and documents after the job/transaction has been completed. 
In the following sections we will look at several areas we can address to manage data retention levels.

```
“CloudOps has seen TA DB growing to 800GB and KAFTA DB grown to over 1TB within 1 year of operations”.
```

Therefore, we should encourage our customers to apply shorter retention periods. Highlight to customers that their KTA platform is not to be treated as a system of record and it is not meant to hold transaction/job data and documents after the job/transaction has been completed. 
In the following sections we will look at several areas we can address to manage data retention levels.

##	Job Data

Currently job data retention policies are configured at a process map level. By default, a process is configured to retain the job data indefinitely. 
![image](https://github.com/user-attachments/assets/295b911e-6124-40b0-9624-80840cf8eb72)  
*Figure 1 Process Map Properties (7.11)*

There have been requests made to Product Engineering to add new features to have greater flexibility to apply retention periods for job data. However, for the time being customers should consider updating these settings to ensure KTA only retains data for the period of time that is absolutely required. We advise all project resources to encourage customers to define the retention periods job data and apply those settings in the process maps.

![image](https://github.com/user-attachments/assets/6c543f92-9a02-49e9-8510-40929c59f122)  
 *Figure 2 Process Map Properties - Retention Applied (7.11)*

Generally, based on observations from previous projects customers tend to retain data for 18 months. However, each customer may have different requirements, so it is important to confirm the requirements with each customer. 

Please also note that if KAFTA is used, the job data (except the job variable data) are also transferred to KAFTA databases. Therefore, customers may choose to have longer retention periods for the Job data in KAFTA while job data gets removed from KTA database.

### APA and other Products

If you are implementing a product or solution such as APA, developed & released by Kofax R&D, then modifying the process map settings may not be viable. You are not permitted to modify the original process maps without breaking supportability. Therefore, at this stage for these kinds of solutions you may continue to use the default settings. If/when more system level configuration options are available to apply retention to job data, you may advise the customers to do so.

####	APA Logs
Ensure APA Log level setting is set to “None” or “Error”. APA Logs provides specific information for each step in the invoice processing especially invoice extraction, validation process. We have observed that APA logs can accumulate and grow the database size quite rapidly if the level is set to “Debug”, “Info” or “Warning”. Therefore, the default setting for APA Log level should be set to “None” or “Error”. In case there is an issue, and you want to diagnose the issue you can set the log level to “Debug” temporarily and revert it back to “None” or “Error” after the investigation.

![image](https://github.com/user-attachments/assets/94449115-7dbb-436c-8bac-68d621f59b7d)  
*Figure 3 APA Log level*

APA also provides a process to purge/delete APA logs. However, this process will delete all logs from the current time (time the job is created) backwards for X number of hours. Therefore, if the customer is interested in keeping recent logs, scheduling this job to run periodically is not appropriate. 
It is recommended to run this job time to time to ensure APA log records are cleared. 

##	Document Data

```
Note: (Update Sep/2024) From TA version 8.0 document retention is set to 6 months by default in the Cloud at a global level for all document types. If you want to overwrite the retention time at global level you can request via TS as indicated in the previous note. The retention periods specified at document type level should override global retention period.
```

Document data is generally the biggest storage consumer since the document binaries are stored in the database. In a KTA Cloud environment document binaries are stored in the Azure Blob storage.
Regardless of where the data is stored it is important that it is not retained in the KTA environment for a long period of time. 
The latest versions of KTA have a number of options to apply retention policies and clean up the document data from the database.
By default, the system is configured to retain document data indefinitely. 

![image](https://github.com/user-attachments/assets/d8f15395-9e2b-4f8e-8c3a-c3fbd490619d)  
*Figure 4 Document Retention Default Settings*

Customers should be advised to enforce retention periods for document data. The best approach is to mark the document as “Finished” at the end of the process and then set the retention policy to delete the document from the system after a given period.
Since different document types can have different retention requirements it is best to set the retention periods at document type level.

![image](https://github.com/user-attachments/assets/53acc66f-8ae8-4946-a56a-3a47cbf406fe)  
*Figure 5 Document Retention Applied*

![image](https://github.com/user-attachments/assets/ec11cc2a-05f8-4f0c-a472-91f1dc11891b)  
*Figure 6 Document Retention Applied - Doc Type*

![image](https://github.com/user-attachments/assets/5abcf819-6621-43ab-8ad3-900bc3989e80)  
*Figure 7 Mark Document as Finished in Process map*


##	Reporting and Insight Analytics Data
Insight project databases is another area where we have seen significant increase of database sizes. Generally, most Cloud customers will have Kofax Analytics for TotalAgility (KAFTA) project deployed by default. And if they are using APA, APA Analytics project will also be deployed by the Cloud team. 
There are several steps that needs to be taken to ensure the Analytics data is maintained and cleaned up periodically.

###	KTA Reporting Retention Settings
By default the document data in the reporting databases are set to be retained for 3650 days. Update this setting to a lower value after discussing with the customer. 

![image](https://github.com/user-attachments/assets/f9937ee1-dbc0-431e-a550-b6c5bae5d329)  
*Figure 8 Reporting Data Retention in KTA Configured*

Also ensure the Retention system tasks’ timeout interval is set to about 15 minutes to ensure the Retention tasks are run without timing out if there is a large chunk of reporting data to be marked for deletion.

![image](https://github.com/user-attachments/assets/6eb71afa-f874-4167-a8c0-09cb5088bf09)  
*Figure 9 KTA Retention System Task Configured*

###	KAFTA
Field data records are a primary contributor to the size of the KAFTA databases. To minimize database growth.

1.	Schedule the Old Data Cleanup Plan to run regularly so that the database growth is managed.
![image](https://github.com/user-attachments/assets/bb00666f-7c3d-4603-ad1a-981c2e57e119)  
*Figure 10 Old Data Cleanup Schedule KAFTA Configured*

2.	Specify the minimum number of days that field data are required in the KAFTA data retention policy.
![image](https://github.com/user-attachments/assets/1949e875-cdef-413c-a86a-ed6648207412)  
*Figure 11 Field Records Retention Period in KAFTA Configured*

Generally, 12-18 months’ worth of data is sufficient. Confirm the retention requirements with the customer. 

Then schedule the "Delete Field Data per Retention Policy" execution plan to run regularly.

![image](https://github.com/user-attachments/assets/4fb7dbc8-7938-427b-9355-38b1dd130c86)  
*Figure 12 Field Data Cleanup Schedule KAFTA*

Schedule the "Delete Field Data per Retention Policy" execution plan to run regularly outside of peak usage hours, such as overnight or on weekends, to avoid any disruptions to normal operations.

For further information refer to KAFTA Best Practices.docx. 

###	APA Analytics

Unfortunately, APA Analytics does not have a proper data cleanup plan. The retention settings need to be applied manually.
At this stage customer will have to monitor this setting periodically and update the value to a desired date. We recommend a date 1 year past from the current date. Kofax project resources should set this value at the time of the implementation after informing and confirming with the customer. 
![image](https://github.com/user-attachments/assets/54e6aa24-cc2c-4936-bd83-228cdcde03b0)  
*Figure 13 APA Analytics Data Retention Threshold*

##	Audit Logs
Audit logs are another source of data contributing to database growth. Depending on the number of users, jobs, and activities the amount of Audit log data can grow significantly. 
Generally, most company policies require audit logs for each system to be retained for a longer period of time. 
In KTA by default, all audit logs are set to be retained indefinitely. 

![image](https://github.com/user-attachments/assets/31e2d7c2-96f7-4fcc-ae1a-94d300442228)  
Figure 14 Audit Logs Default Settings
Customers should be encouraged to apply a retention period for audit logs. Generally, 5-6 years’ worth of data is sufficient. 
 
![image](https://github.com/user-attachments/assets/5b467e36-e9e9-4f00-b2ac-152aa4829c49)  
Figure 15Retention Policy Applied

#	License Thresholds
Monitoring license volumes/counts is another aspect that needs attention to ensure uninterrupted availability of our cloud services.
When a KTA cloud tenant is deployed the Kofax CloudOps team will allocate a number of licenses. Customers and Kofax project resources must be aware of these allocations and ensure necessary license thresholds and notifications are configured in KTA to send alerts regarding the usage of the licenses. 
By default, license thresholds are not set in KTA.
 
![image](https://github.com/user-attachments/assets/fc2e31c6-2f55-4973-a39c-c0b1052569a0)  
Figure 16 License Thresholds Default Settings

To enable license thresholds, update the settings in KTA designer and configure the exception maps to notify a nominated customer mailbox.

If Kofax Professional Services are engaged by the customer for the solution implementation following license thresholds and alerts must be configured as a standard practice. If a customer is carrying out the implementation by themselves or using a partner, Kofax Account Managers/PMs must inform and guide customers to carry out the necessary configurations.

 
![image](https://github.com/user-attachments/assets/753e6b2e-a0c1-41b4-9ca9-6c5c858947fa)  
Figure 17 License Thresholds Configured

 
![image](https://github.com/user-attachments/assets/531c2191-3880-4ce7-8e5d-e9060d6e72f8)
Figure 18 License Alerts Configured

#	Monitoring & Exception Handling
To ensure customers are alerted to any failures in the KTA Cloud environment there are several monitoring options that should be configured. Often customers are unaware of these settings. Kofax PS, if engaged, must ensure the customer is made aware of these settings and configure them.
The monitoring options are outlined in here: Kofax Cloud Monitoring v1.1.pdf

#	Online Learning
If the solutions that are being deployed in KTA Cloud have Online Learning turned ON, we need to ensure that the customer understands how online learning works and on-going maintenance it requires.
Firstly, ensure that the online learning document limit is not set to a large number. For example, in the APA solution the “Maximum documents stored for import” is set to 10000. We have experienced that having a large number of documents can cause issues when trying to download and train the online learning samples. Retraining and conflict resolution in TD will also be cumbersome when there are too many documents. 
Therefore it is recommended to set the limit to 2000 or less.

![image](https://github.com/user-attachments/assets/374e3fb8-95c9-4816-a2b1-be024b4abf3e)  
Figure 19 Classification Online Learning Settings
 
![image](https://github.com/user-attachments/assets/485786d1-2ce9-4886-a4ae-aac70a776ba5)  
Figure 20 Extraction Online Learning Settings

Secondly customers should be informed and trained to ensure that the online learning maintenance is carried out periodically as per best practices. Customer should be informed of the consequences of not maintaining the Online Learning, e.g. extraction/classification performance may degrade and accuracy will suffer.

See more information about Online Learning here: TD Online Learning Best Practise Guide 03052023.pdf

6	User Access Management
Typically, when implementing KTA in on-premises customers would have a dedicated team to support the administration of the KTA application. However, with KTA cloud the customer’s expectation and understanding about their responsibilities might be different. We have observed that customers are not taking enough ownership of managing and administering the KTA platform as they expect it to be a fully managed SaaS service. This expectation is wrong, and we should advise customers to appoint dedicated resources to manage and administer the KTA platform.
User Access Management is an area that the appointed customer application administrators should be responsible. In the initial stages of KTA project implementation, Kofax PS resources may have administrative access to the KTA designer to create user accounts and assign permissions in the Dev environment. But customer appointed application administrators should be managing the user provisioning and permissions going forward in the other environments. It is expected that the customer resources may not be trained to manage the user provisioning and permission management at the beginning. However, as a pre-requisite, customer resources should complete KTA product training and be prepared to take on the responsibility. During the implementation in the initial stage’s customer resources may shadow Kofax PS resources get familiar with the user provisioning and permission managing activities.
The goal is to put the complete onus of managing access to the KTA platform onto the customer.

#	Product Training
Even though the KTA Cloud environment is hosted by Kofax, it is the customer’s responsibility to manage and administrate the platform and any solutions configured on it. 
Kofax PS may be engaged in the initial stages of the implementation process, however Kofax PS will not be responsible for managing the platform post implementation. Therefore, customers must have dedicated resources to support these administrative tasks. To be able to support management of the platform, customers must complete Kofax product training and acquire the necessary skills.

Kofax PS should ensure customers do take up the recommended training courses and understand their responsibilities.
Suggested training courses:

•	Kofax Course Details - TotalAgility Administrator  
•	Kofax Course Details - TotalAgility Citizen Developer  
•	Kofax Course Details - TotalAgility Professional Developer  
•	Kofax Course Details - TotalAgility Master Class (Instructor-led) (for advance users)  
•	Kofax Course Details - AP Agility Solution Developer  

#	Checklist
Ensure the following checklist is completed and signed off by customer.  


[KTA Cloud Implementation Checklist.docx](https://github.com/user-attachments/files/17037454/KTA.Cloud.Implementation.Checklist.docx)


