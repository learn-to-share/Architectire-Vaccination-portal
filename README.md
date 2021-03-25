# Architecture-Vaccination-Portal


A refernce AWS architecture to design  COVID-19 Vaccination portal.

**Functional Requirements:**

1. Users should be able to register & schedule for vaccination

2. Users should be able to download vaccination certificate

**Non-Functional Requirements:**

1. Users should be able to select clinic of their choice

2. Users should receive OTP confirming the appointment 

3. Users should receive reminder 1 day before the appointment

4. users can re-schedule the appointment 

**Assumptions:**

1. No user Signup is required.

**AWS Service Selection:**

--> Multiple users are expected to use the portal whenever they are eligible. Hence a ELB is required to load balance the traffic. SSL termination on ELB.

--> Web and APP logic can be run on the same EC2 instances as this is not a app logic intensive workload. Can maintain at least 1-2 resrved instances across 2 availability zones with 1 year reservation. Can rely on spot/on-demand instances for unexpected traffic. 

--> App logic should be able to handle when any spot instance is terminated.

--> Any RDS DB is required to store the user information and vaccination records persistently. RDS Mysql or Aurora can be considered with one Standby for DR scenario.

--> 2 Separate tables recommened. 1 for storing user details and schedule. 2d for storing clinic etc details.

DB Schema:

 **User Table**:
 
 ![image](https://user-images.githubusercontent.com/39849388/112439619-4fdcc500-8d84-11eb-8a70-a7441dff6687.png)

 
 **Cert_record Table:**

![image](https://user-images.githubusercontent.com/39849388/112439730-70a51a80-8d84-11eb-9ef0-ae5651922c00.png)

 
 --> SNS topic is required to send notification to users

--> A lambda with cron set everyday at fixed timings to send reminders for next day appointments.

--> Amazon quicksights can be used to display dashboard on the overall program  status(Optional)

--> A separate URL path for clinics to upload the reports daily in CSV format. App logic to prcoess them and update DB. 


**Architecture Diagram:**

![image](https://user-images.githubusercontent.com/39849388/112437329-c9bf7f00-8d81-11eb-9951-f6902aaca0d0.png)



**Work Flow:**

1. Users will access web portal and the request will be sent to DNS via Route 53. Route 53 will forward the request to ELB.

2. ELB will do SSL Termination on it and will forward the request to EC2 instances.

3. EC2 instances will process the request and send the data to DB

4. App on EC2 will send a confirmation to user via Web Portal and via SMS/Email using SNS service. Respective Clinic will be notified. 

5. Lambda function will be set with a cron job to run everyday at fixed time to send reminders for the next day scheduled users. And another Lambda function to send confirmed users list to respective clinics.

6. By EOD(Can be made online also using the same portal with different URL path) all clinics will upload their records to the web portal using a separte URL path. EC2 instanes will process this data and update DB.

7. Users can download their Vaccination Certifiates from the portal. App on EC2 will generate a certificate dynamically using the records in the DB.



