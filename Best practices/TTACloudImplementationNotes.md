#Purpose
This document aims to provide some guidelines for Kofax KTA Cloud implementations.
At the time this document was written, Kofax Professional Services have completed several KTA Cloud implementations. There have been several issues and observations from the KTA Cloud implementations thus far. This document aims to capture some of the guidelines for Kofax resources who are working with KTA Cloud to overcome/avoid those issues in future implementations.

#Data Retention

Note: (Update Sep/2024) From TA version 8.0 some retention settings are managed by TA CloudOps. Professional Services shall submit the retention periods required via a TS case. Use the spreadsheet provided to specify the retention periods required.

Retention policies are important in any implementation, whether it's on-premises, private cloud, or public cloud. However, when it comes to a SaaS (Software as a Service) environment it is even more important to apply the appropriate retention polices and configurations to avoid large amounts of data accumulating in the SaaS infrastructure.
Exponential data growth results in additional costs, maintenance overheads and performance issues and ultimately customer dissatisfaction.
Therefore, we should encourage our customers to apply shorter retention periods. Highlight to customers that their KTA platform is not to be treated as a system of record and it is not meant to hold transaction/job data and documents after the job/transaction has been completed. 
In the following sections we will look at several areas we can address to manage data retention levels.

**“CloudOps has seen TA DB growing to 800GB and KAFTA DB grown to over 1TB within 1 year of operations”.**
