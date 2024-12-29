# Day 7: Oh, no. I'M SPEAKING IN CLOUDTRAIL!

### Monitoring in an AWS Environment
Care4Wares' infrastructure runs in the cloud, so they chose **AWS** as their Cloud Service Provider (CSP). Instead of their workloads running on physical machines on-premises, they run on virtualised instances in the cloud. These instances are (in AWS) called EC2 instances (Amazon Elastic Compute Cloud). A few members of the Wareville SOC aren't used to log analysis on the cloud, and with a change of environment comes a change of tools and services needed to perform their duties. Their duties this time are to help Care4Wares figure out what has happened to the charity's funds; to do so, they will need to learn about two AWS services: CloudWatch and CloudTrail.

### Intro to JQ
It was mentioned that CloudTrail logs were JSON-formatted. When ingested in large volumes, this machine-readable format can be tricky to extract meaning from, especially in the context of log analysis. The need then arises for something to help us *transform and filter that JSON data* into meaningful data. That's exactly what JQ is (and does!). Similar to command line tools like sed, awk and grep, JQ is a lightweight and flexible command line processor that can be used on JSON.

### The Peculiar Case of Care4Wares’ Dry Funds
The Care4Wares charity drive gave us the following info regarding this incident:

_We sent out a link on the 28th of November to everyone in our network that points to a flyer with the details of our charity. The details include the account number to receive donations. We received many donations the first day after sending out the link, but there were none from the second day on. I talked to multiple people who claimed to have donated a respectable sum. One showed his transaction, and I noticed the account number was wrong. I checked the link, and it was still the same. I opened the link, and the digital flyer was the same except for the account number._

McSkidy recalls putting the digital flyer, **wareville-bank-account-qr.png**, in an Amazon AWS S3 bucket named **wareville-care4wares**. We'll assist McSkidy and start by finding out more about that link. Let’s first review the information that we currently have to start the investigation:

- The day after the link was sent out, several donations were received.
- Since the second day after sending the link, no more donations have been received.
- A donator has shown proof of his transaction. It was made 3 days after he received the link. The account number in the transaction was not correct.
- McSkidy put the digital flyer in the AWS S3 object named **wareville-bank-account-qr.png** under the bucket **wareville-care4wares**.
- The link has not been altered.
#### **Glitch Did It**
Let’s use JQ to filter the log for events related to the **wareville-bank-account-qr.png** S3 object. The goal is to use the same elements to filter the log file using JQ and format the results into a table to make it more readable. According to McSkidy, the logs are stored in the `~/wareville_logs` directory.

1. To start, open the Terminal and enter the two commands below:

<img width="350" alt="THM AoC 2024 Day 7-2" src="https://github.com/user-attachments/assets/fd40d9cc-1501-4e51-904c-23444f4536e5" />

2. We will focus first on the **cloudtrail_log.json** for this investigation. Let's start investigating the CloudTrail logs by executing the command: `jq -r '.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares")' cloudtrail_log.json`.

<img width="444" alt="THM AoC 2024 Day 7-3" src="https://github.com/user-attachments/assets/0db21c10-dd31-4d7b-a5e8-efe5798b4c47" />

As you can see in the command output, we were able to trim down the results since all of the entries are from S3. 

3. Let's refine the output by selecting the significant fields. Execute: `jq -r '.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares") | [.eventTime, .eventName, .userIdentity.userName // "N/A",.requestParameters.bucketName // "N/A", .requestParameters.key // "N/A", .sourceIPAddress // "N/A"]' cloudtrail_log.json`

<img width="434" alt="THM AoC 2024 Day 7-4" src="https://github.com/user-attachments/assets/f1945f6a-206e-49ac-a7ce-b16d6808fb01" />

As you can see in the results, we could focus on the notable items, but our initial goal is to render the output in a table to make it easy to digest. 

4. Let's upgrade our command with additional parameters. Execute the command: `jq -r '["Event_Time", "Event_Name", "User_Name", "Bucket_Name", "Key", "Source_IP"],(.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares") | [.eventTime, .eventName, .userIdentity.userName // "N/A",.requestParameters.bucketName // "N/A", .requestParameters.key // "N/A", .sourceIPAddress // "N/A"]) | @tsv' cloudtrail_log.json | column -t`

<img width="746" alt="THM AoC 2024 Day 7-5" src="https://github.com/user-attachments/assets/aeb13b79-2952-498c-87d7-aa46d3edac49" />

Now that we have crafted a JQ query that provides a well-refined output, let’s look at the results and observe the events. Based on the columns, we can answer the following questions to build our assumptions:

- How many log entries are related to the **wareville-care4wares** bucket?
- Which user initiated most of these log entries?
- Which actions did the user perform based on the **eventName** field?
- Were there any specific files edited?
- What is the timestamp of the log entries?
- What is the source IP related to these log entries?

Looking at the results, 5 logged events seem related to the **wareville-care4wares** bucket, and almost all are related to the user **glitch**. Aside from listing the objects inside the bucket ("ListObjects" events), the most notable detail is that the user glitch uploaded ("PutObject") the file **wareville-bank-account-qr.png** on November 28th. This seems to coincide with the information we received about no donations being made 2 days after the link was sent out.

McSkidy is sure there was no user glitch in the system before. There is no one in the city hall with that name, either. The only person that McSkidy knows with that name is the hacker who keeps to himself. McSkidy suggests that we look into this anomalous user.

#### **McSkidy Fooled Us?**

McSkidy wants to know what this anomalous user account has been used for, when it was created, and who created it. We can focus our analysis on the following questions:

- What event types are included in these log entries?
- What is the timestamp of these log entries?
- Which IPs are included in these log entries?
- What tool/OS was used in these log entries?

1.  Enter the command to see all the events related to the anomalous user:

<img width="752" alt="THM AoC 2024 Day 7-6" src="https://github.com/user-attachments/assets/9a29d68c-7a6e-438b-b22d-49244ec68f99" />

The results show that the user glitch mostly targeted the S3 bucket. The notable event is the **ConsoleLogin** entry, which tells us that the account was used to access the AWS Management Console using a browser.

2. We still need information about which tool and OS were used in the requests. Let's view the **userAgent** value related to these events using the following command.

<img width="779" alt="THM AoC 2024 Day 7-7" src="https://github.com/user-attachments/assets/7699e53b-81b8-4ce9-8404-27237cb68563" />

There are two **User_Agent** values included in all log entries related to the **glitch** user:
- `[S3Console/0.4, aws-internal/3 aws-sdk-java/1.12.750 Linux/5.10.226-192.879.amzn2int.x86_64 OpenJDK_64-Bit_Server_VM/25.412-b09 java/1.8.0_412 vendor/Oracle_Corporation cfg/retry-mode/standard]`
	- This is the userAgent string for the internal console used in AWS. It doesn’t provide much information.
- `[Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36]`
	- This userAgent string provides us with 2 pieces of interesting information.
	- The anomalous account uses a Google Chrome browser within a Mac OS system.

An experienced attacker can forge these values, but we should not dismiss this information. It can be valuable when comparing different log entries for the same user. We will park the current information for now, let's gather more information to connect the dots.

3. The next interesting event to look for is who created this anomalous user account. We will filter for all IAM-related events, and this can be done by using the select filter `.eventSource == "iam.amazonaws.com"`. Let's execute the command below.

<img width="802" alt="THM AoC 2024 Day 7-8" src="https://github.com/user-attachments/assets/047754e3-5904-42ed-a735-0ba398cf193b" />

4. Now we'll try to answer the following questions:
	- What Event Names are included in the log entries?
	- What user executed these events?
	- What is this user’s IP?

Based on the results, there are many ListPolicies events. By ignoring these events, it seems that the most significant IAM activity is about the user **mcskidy** invoking the **CreateUser** action and consequently invoking the **AttachUserPolicy** action. The source IP where the requests were made is **53.94.201.69**. Remember that it is the same IP the anomalous user glitch used.

5. Let’s have a more detailed look at the event related to the **CreateUser** action by executing the command below:

<img width="670" alt="THM AoC 2024 Day 7-9" src="https://github.com/user-attachments/assets/e009b44d-5f48-4ba7-a118-d98beacdc22b" />

Based on the request parameters of the output, it can be seen that it was the user, **mcskidy**, who created the anomalous account.

6. Now, we need to know what permissions the anomalous user has. It could be devastating if it has access to our whole environment. We need to filter for the **AttachUserPolicy** event to uncover the permissions set for the newly created user. This event applies access policies to users, defining the extent of access to the account. Let's filter for the specific event by executing the command below.

<img width="680" alt="THM AoC 2024 Day 7-10" src="https://github.com/user-attachments/assets/e71f44d5-2b3a-414a-a3eb-45687864f1cd" />

McSkidy is baffled by these results! She knows that she did not create the anomalous user and did not assign the privileged access. She also doesn’t recognise the IP address involved in the events and does not use a Mac OS; she only uses a Windows machine. All this information is different to the typical IP address and machine used by McSkidy, so she wants to prove her innocence and asks to continue the investigation.

#### **Logs Don’t Lie**

McSkidy suggests looking closely at the IP address and operating system related to all these anomalous events. 

1. Let's use the following command below to continue with the investigation:

<img width="677" alt="THM AoC 2024 Day 7-11" src="https://github.com/user-attachments/assets/77bad526-ad5a-4768-b2b5-1bf8c68ad027" />

Based on the command output, three user accounts (**mcskidy**, **glitch**, and **mayor_malware**) were accessed from the same IP address. The next step is to check each user and see if they always work from that IP.

2. Let’s focus on each user and see if they always work from that IP. Enter the following command, and replace `PLACEHOLDER` with the username: `jq -r '["Event_Time","Event_Source","Event_Name", "User_Name","User_Agent","Source_IP"],(.Records[] | select(.userIdentity.userName=="PLACEHOLDER") | [.eventTime, .eventSource, .eventName, .userIdentity.userName // "N/A",.userAgent // "N/A",.sourceIPAddress // "N/A"]) | @tsv' cloudtrail_log.json | column -t -s $'\t'`


3. We can focus our investigation on the following questions:
	- Which IP does each user typically use to log into AWS?
	- Which OS and browser does each user usually use?
	- Are there any similarities or explicit differences between the IP addresses and operating systems used?

Based on the results, we have proven that McSkidy used a different IP address before the unusual authentication was discovered. Moreover, all evidence seems to point towards another user after correlating the IP address and User_Agent used by each user. Who do you think it could be? McSkidy has processed all the investigation results and summarised them below:

- The incident starts with an anomalous login with the user account **mcskidy** from IP **53.94.201.69**.
- Shortly after the login, an anomalous user account **glitch** was created.
- Then, the **glitch** user account was assigned administrator permissions.
- The **glitch** user account then accessed the S3 bucket named **wareville-care4wares** and replaced the **wareville-bank-account-qr.png** file with a new one. The IP address and User_Agent used to log into the **glitch, mcskidy**, and **mayor_malware** accounts were the same.
- the User_Agent string and Source IP of recurrent logins by the user account **mcskidy** are different.

#### **Definite Evidence**
McSkidy suggests gathering stronger proof that that person was behind this incident. Luckily, Wareville Bank cooperated with us and provided their database logs from their Amazon Relational Database Service (RDS). They also mentioned that these are captured through their CloudWatch, which differs from the CloudTrail logs as they are not stored in JSON format. For now, let’s look at the bank transactions stored in the `~/wareville_logs/rds.log` file.

Since the log entries are different from the logs we previously investigated, McSkidy provided some guidance on how to analyse them. According to her, we can use the following command to show all the bank transactions: `grep INSERT rds.log`

From the command above, McSkidy explained that all INSERT queries from the RDS log pertain to who received the donations made by the townspeople. Given this, we can see in the output the two recipients of all donations made within November 28th, 2024:

<img width="542" alt="THM AoC 2024 Day 7-14" src="https://github.com/user-attachments/assets/62cabf9f-6c03-43a5-98d5-a61e158fd144" />

As shown above, the Care4wares Fund received all the donations until it changed into a different account at a specific time. The logs also reveal who received the donations afterwards, given the account owner's name. With all these findings, McSkidy confirmed the assumptions made during the investigation of the S3 bucket since the sudden change in bank details was reflected in the database logs. The timeline of events collected by McSkidy explains the connection of actions conducted by the culprit:

<img width="416" alt="THM AoC 2024 Day 7-13" src="https://github.com/user-attachments/assets/2b5e213d-2950-4cd5-b3ae-3a664777755d" />



### Answers

1. What is the other activity made by the user glitch aside from the ListObject action? **PutObject**.
3. Based on the eventSource field, what AWS service generates the ConsoleLogin event? **signin.amazonaws.com**.
4. When did the anomalous user trigger the ConsoleLogin event? **2024-11-28T15:21:54Z**.
5. What was the name of the user that was created by the mcskidy user? **glitch**.
6. What type of access was assigned to the anomalous user? **AdministratorAccess**.
7. Which IP does Mayor Malware typically use to log into AWS? **53.94.201.69**.
8. What is McSkidy's actual IP address? **31.210.15.79**.
9. What is the bank account number owned by Mayor Malware? **2394 6912 7723 1294**.
