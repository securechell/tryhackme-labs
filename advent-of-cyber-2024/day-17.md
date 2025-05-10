# Day 17: He analyzed and analyzed till his analyzer was sore!

**Learning Objectives**
- Learn how to extract custom fields in Splunk
- Learn to create a parser for the custom logs
- Filter and narrow down the search results using Search Processing Language (SPL)
- How to investigate in Splunk

### Investigation Time
1. It's time to fire up Splunk. Open up the link in the browser and click on **Search & Reporting** on the left.
2. On the next page, type `index=*` in the search bar to show all ingested logs. Note that we will need to select **All time** as the time frame from the drop-down on the right of the search bar.
![image](https://github.com/user-attachments/assets/5a3e68f3-de3d-4807-80ae-a153d2da640e)

3. After running the query, we will be presented with two separate datasets pre-ingested to Splunk. We can verify this by clicking on the **sourcetype** field in the fields list on the left of the page.
![image](https://github.com/user-attachments/assets/84a4b3a1-1f3b-44a6-a84f-c6d0cccff38b)

The two datasets are as follows:
- `web_logs`: This file contains events related to web connections to and from the CCTV web server.
- `cctv_logs`: This file contains information about the CCTV application access logs.

Let's explore the logs and investigate the attack on our CCTV servers to identify the culprit, who got unauthorised access to the server and deleted the CCTV streams.

#### Examining CCTV Logs
Let's start our investigation by examining the **CCTV logs**. To do so, we can either click on the corresponding value for the `sourcetype` field, or type the following query in the search bar: `index=* sourcetype=cctv_logs`
![image](https://github.com/user-attachments/assets/ee163bd6-91dd-4a6e-8042-51ce667f19f2)

### Understanding the Problem
After examining the logs, we can figure out the following main issues:
- Logs are not parsed properly by Splunk.
- Splunk does not consider the actual timeline of the event; instead, it uses only the ingestion time.

### Fixing the Problem
Before analysing and investigating the logs, we must extract the relevant fields from them and adjust the timestamp.

The provided logs were generated from a custom log source, so Splunk could not parse the fields properly.

1. **Extract New Field:** Click on the **Extract New Fields** option, located below the fields list on the left of the page.
![image](https://github.com/user-attachments/assets/44b154d0-1c97-40a5-8bcc-49ee9d8cf4af)

2. **Select Sample Event:** Let's select the very first sample event and click on the green **Next** button at the top of the page.
3. **Select Method:** There are two options for extracting the fields:
	- using Regular Expressions
	- using Delimiters. 
	In this exercise, we will extract fields using Regular Expressions. Select this option and then click **Next**.
4. **Select Fields:** Now, to select the fields in the logs that we want to extract, we simply need to highlight them in the sample log. Splunk will autogenerate the regex (regular expression) to extract the selected field.
![image](https://github.com/user-attachments/assets/4f387873-f282-4363-947d-bfbb7665c9f0)

![image](https://github.com/user-attachments/assets/3ee57c03-80ce-44dc-b0e0-2a63ca9b05c0)

We'll assign an appropriate name to each of the extracted fields based on the table below:
![image](https://github.com/user-attachments/assets/58f4b8d4-8c35-45da-b7ff-1164c05a3f3a)

As evident from the preview section, by selecting the fields, Splunk creates a regular expression to extract that field from all the events.

All the extracted fields will be displayed in the Preview tab, as shown below:
![image](https://github.com/user-attachments/assets/a6f36888-82b7-40c1-9f68-158611f30c4e)

It is important to note that some of the logs may have a different format, and they may not be parsed using the parser we created above. We may have to re-extract the fields from those events. We will get back to fixing this issue later.  

We can click on each extracted field to check the extracted values. When we're satisfied with the extracted values, we can click **Next**.

5. **Validate:** In the next step, we will see a green tick mark next to the sample logs to indicate the correct extraction of the fields, or a red cross sign to signal an incorrect pattern, as shown below. Click **Next**.
![image](https://github.com/user-attachments/assets/01d06905-66c4-418f-b469-056907479f24)

6. **Save and Analyse:** After validating that the extracted fields are correct, the next step is saving and analysing the logs.
![image](https://github.com/user-attachments/assets/da0237eb-5e10-495f-8c51-6de9f8a903ef)

This tab shows us the regular expression created, the fields extracted, and the sample event that contains the fields we wanted to extract. Save this session by clicking on the green **Finish** button and move on to the search tab to search the logs. To do so, we can click **Explore the fields I just created in Search** on the next page.
![image](https://github.com/user-attachments/assets/c7e06e71-3038-4134-bb55-5e21bf20b8e8)

We can verify that we successfully extracted the custom fields from the logs by clicking on any of our custom fields in the list on the left of the page. For example, if we click on the `UserName` field, we'll be presented with all the different values that have been extracted from the logs for this field.
![image](https://github.com/user-attachments/assets/895f5c76-f874-4d83-9901-e187fe800b5e)

It also appears that some fields have not been parsed exactly as we expected:
![image](https://github.com/user-attachments/assets/b32d2611-8e79-4c56-8a65-b94da90a746a)

### Improving the Field Extraction
As previously mentioned, some of the logs are a bit different from the ones we used as a baseline for the field extraction. Some of the log formats that our parser could not pick are mentioned below:
![image](https://github.com/user-attachments/assets/c08c5b1e-86f6-44f1-ac1c-0d821357fa7b)

It is important to note that, there can be various ways to achieving our goal of fixing the parser. We will try one of the methods, as covered in coming steps.

1. **Removing the Fields Extraction:** Go to Settings -> Fields
2. **Field Extraction:** Click on the Field extractions tab; it will display all the fields extracted.
3. **Delete the Regex Pattern:** This tab will display all the patterns/fields extracted so far in Splunk. We can look for the `cctv` related pattern in the list, or simply search `cctv` in the search bar, and it will display our recently created pattern. Once the right pattern is selected, click on the **Delete** button.
![image](https://github.com/user-attachments/assets/67ad9701-d1cb-4fca-9749-90154a458077)

Why we are deleting this previously created pattern? Well, this regex picks fields from some logs and leave behind other logs, which may be vital for our investigation.  
Our goal is to create one generic regular expression, that works on almost all events.  

4. **Open Field Extractor:** Next, click on the **Open Field Extractor** button, and it will take us to the same tab, where we can extract the fields again.
5. **Update the Regex:** This time, after selecting the right source type as `cctv_logs`, and time range as `All Time`, click on **I prefer to write the regular expression myself**.
![image](https://github.com/user-attachments/assets/fe62c174-1e7b-4bc0-b3c7-89d5de6c86c5)

In the next tab, enter the regex `^(?P<timestamp>\d+\-\d+\-\d+\s+\d+:\d+:\d+)\s+(?P<Event>(Login\s\w+|\w+))\s+(?P<user_id>\d+)?\s?(?P<UserName>\w+)\s+.*?(?P<Session_id>\w+)$` and select **Preview**.
![image](https://github.com/user-attachments/assets/077c4636-36e7-470f-89ad-d15d9efa652e)

This regex will fix the field parsing pattern and extract all needed fields from the logs. Click **Save** and on the next page, select **Finish**.

On the next page, once again, click on the **Explore the fields I just created in Search**.

Now that we can observe that all fields are being extracted as we wanted, let's start investigating the logs.  

### Investigating the CCTV Footage Logs
Now that we have sanitised and properly parsed the logs, it's time to examine them and find the culprit.

1. **Summary of the CCTV Feed**

After examining the CCTV feed logs, we can create a mental picture of the information these logs provide us. A brief summary of these logs is:
- These logs contain the successful and failed login attempts from various users.
- They contain a few failed login attempts, which looks suspicious.
- They contain information about the CCTV footage being watched and downloaded.

2. **Event Count by Each User**

Let's use the following search query to see the count of events by each user: `index=cctv_feed | stats count(Event) by UserName`

We can easily visualise this data by first clicking on **Visualization** below the search bar, then change the visualisation type from **Bar Chart** to **Pie Chart**.
![image](https://github.com/user-attachments/assets/ae159743-09be-4c37-b76d-40f8e3fce90b)

3. **Summary of the Event Count**

We can create a summary of the event count to see what activities were captured in the logs using the following query: `index=cctv_feed | stats count by Event`

Splunk will automatically display the previously selected **Pie Chart** type of visualisation.

4. **Examining Rare Events**

Using the following search query, let's look at the events with fewer occurrences in the event field to see if we can find something interesting: `index=cctv_feed | rare Event`
![image](https://github.com/user-attachments/assets/dbb1011d-4ea2-466a-960f-3e534d9a9776)

It looks like we have a few attempts to delete the recording and a few failed login attempts. This means we have a clue. Let's now examine the failed login attempts first: `index=cctv_feed *failed* | table _time UserName Event Session_id`
![image](https://github.com/user-attachments/assets/d7da085c-8b4c-4d12-910b-e7a8b8b65980)

We found some failed login attempts against a few users, but one thing remains constant: the Session_id.

5. **Narrowing Down Our Investigation**

Let's narrow down our results to see what other events are associated with this Session_id: `index=cctv_feed *rij5uu4gt204q0d3eb7jj86okt* | table _time UserName Event Session_id`
![image](https://github.com/user-attachments/assets/e9e288b4-2e5b-48d4-8eec-9fbd21d9acd0)

Let's see how many events related to the deletion of the CCTV footage were captured: `index=cctv_feed *Delete*`
![image](https://github.com/user-attachments/assets/b1b4ae3c-46a4-4cc2-a757-bb52a24636fc)

Good. We have some comprehensive information about the attacker and his notorious activities.

6. **Correlating With the Web Logs**

Let's use the information extracted from the earlier investigation (`index=cctv_feed *rij5uu4gt204q0d3eb7jj86okt* | table _time UserName Event Session_id`) and correlate it with the web logs.

7. **Suspicious IP Address**

During the examination, it is observed that only one IP address **10.11.105.33** is associated with the suspicious session ID.

Identify the footprint associated with the session ID: `index=web_logs *rij5uu4gt204q0d3eb7jj86okt*`
![image](https://github.com/user-attachments/assets/1e552ffa-0d1d-40e7-b6af-19829e94269a)

Let's narrow down the search to show results associated with the IP address found earlier: `index=web_logs clientip="10.11.105.33`. It is also important to note that, in this case, the details about the session IDs are found in the field status.
![image](https://github.com/user-attachments/assets/876937d6-3329-4dca-b5cd-a7e2743d4149)

It looks like two more Session IDs were associated with the IP address found earlier. Let's create a search to observe what kind of activities were captured associated with the IP and these session IDs: `index=web_logs clientip="10.11.105.33" | table _time clientip status uri ur_path file`
![image](https://github.com/user-attachments/assets/be7434d9-8871-4d9d-87b9-60975e4f0488)

Looking closely, we can see logout events when the session ID was changed. Can we correlate these session IDs in the cctv_feeds logs and see if we can find any evidence?

8. **Connecting the Dots**

Let's go back to `cctv_feed` and use these session IDs associated with the IP address, as shown: `index=cctv_feed *lsr1743nkskt3r722momvhjcs3*`
![image](https://github.com/user-attachments/assets/e116d372-f624-49eb-8f8a-9a60c0e15191)

Great, we were able to locate the user name associated with the attack. Now that we have identified the user, let's summarise our investigation.

From the output, it seems the following was the timeline of the attack:

- Attacker bruteforce attempt on various accounts.
- There was a successful login after the failed attempts.
- Attacker watched some of the camera streams.
- Multiple camera streams were downloaded.
- Followed by the deletion of the CCTV footage.
- The web logs had an IP address associated with the attacker's session ID.
- We found two other session IDs associated with the IP address.
- We correlated back to the cctv_feed logs to find the traces of any evidence revolving around those session IDs, and found the name of the attacker.

### Answers
1. Extract all the events from the cctv_feed logs. How many logs were captured associated with the successful login? **642**.
![image](https://github.com/user-attachments/assets/90d2e58f-5293-4234-9bf8-22981c31dc54)

2. What is the Session_id associated with the attacker who deleted the recording? **rij5uu4gt204q0d3eb7jj86okt**.
![image](https://github.com/user-attachments/assets/d88ed8b7-c49a-49cf-8a02-2a0139a8b11c)

3. What is the name of the attacker found in the logs, who deleted the CCTV footage? **mmalware**.
