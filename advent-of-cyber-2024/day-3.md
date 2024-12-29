# Day 3: Even if I wanted to go, their vulnerabilities wouldn't allow it.

**Learning Objectives**
- Learn about Log Analysis and tools like ELK.
- Learn about KQL and how it can be used to investigate logs using ELK.
- Learn about RCE (Remote Code Execution), and how this can be done via insecure file upload.

## 🔵 OPERATION BLUE 🔵
In this section of the lesson, we will take a look at what tools and knowledge is required for the _blue_ segment, that is the investigation of the attack itself using tools which enable is to analyse the logs. 

For the first part of Operation Blue, we will demonstrate how to use **ELK** to analyse the logs of a demonstration web app - WareVille Rails.

### Log Analysis & Introducing ELK
Log analysis is crucial to blue-teaming work. Analysing logs can quickly become overwhelming, especially if you have multiple devices and services. **ELK**, or **Elasticsearch, Logstash, and Kibana**, combines data analytics and processing tools to make analysing logs much more manageable. ELK forms a dedicated stack that can aggregate logs from multiple sources into one central place.

The first part of today's task is to investigate the attack on Frosty Pines Resort's Hotel Management System to see what it looks like to a blue teamer.

### Using ELK

1. We will use **Kibana's Discover** interface to review Apache2 logs. To access this, simply click on the **hamburger menu** -> **Discover**.
2. We will need to select the collection (group of logs) that is relevant to us. For this stage of Operation Blue, we will be reviewing the logs present within the "wareville-rails" collection. To select this collection, click on the dropdown on the left of the display.

![image](https://github.com/user-attachments/assets/011e67af-a276-4b36-b8a0-a9b5b7b062cd)

3. Once you have done this, you will be greeted with a screen saying, "No results match your search criteria". This is because no logs have been ingested within the last 15 minutes.
4. Change the date and time. Click the text located on the right side of the calendar icon. Select "**Absolute"** to select the start/end dates and times. For the WareVille Rails collection, we will need to set the start time to **October 1 2024 00:00:00**, and the end time to **October 1 23:30:00**

### Basics of the Kibana Discover UI

![image](https://github.com/user-attachments/assets/90f354d6-1fd4-4244-9a85-525d92430d95)

1. **Search Bar:** Here, we can place our search queries using KQL
2. **Index Pattern:** An index pattern is a collection of logs. This can be from a specific host or, for example, multiple hosts with a similar purpose (such as multiple web servers). In this case, the index pattern is all logs relating to "wareville-rails"
3. **Fields:** This pane shows us the fields that Elasticsearch has parsed from the logs. For example, timestamp, response type, and IP address.
4. **Timeline:** This visualisation displays the event count over a period of time
5. **Documents (Logs):** These entries are the specific entries in the log file
6. **Time Filter:** We can use this to narrow down a specific time frame (absolute). Alternatively, we can search for logs based on relativity. I.e. "Last 7 days".

### Kibana Query Language (KQL)
**KQL** is an easy-to-use language that can be used to search documents for values. For example, querying if a value within a field exists or matches a value. If you are familiar with Splunk, you may be thinking of SPL (Search Processing Language).

For example, the query to search all documents for an IP address may look like `ip.address: "10.10.10.10"`.

The table below contains a mini-cheatsheet for KQL syntax that you may find helpful in today's task.

![image](https://github.com/user-attachments/assets/67adc991-1fb7-4f0e-8a91-ec4bae60ab71)

### Investigating a Web Attack With ELK
#### Scenario:
Thanks to our extensive intrusion detection capabilities, our systems alerted the SOC team to a web shell being uploaded to the WareVille Rails booking platform on Oct 1, 2024. Our task is to review the web server logs to determine how the attacker achieved this.

1. Let's change the time filter to show events for the day of the attack, setting the start date and time to "**Oct 1, 2024 @ 00:00:00.000**" and the end date and time to "**Oct 2, 2024 @ 00:00:00.000**". You will see the logs have now populated within the display.

2. An incredibly beneficial feature of ELK is that we can filter out noise. A web server (especially a popular one) will likely have a large number of logs from user traffic—completely unrelated to the attack. Using the fields pane, we can click on the "**+**" and "**-**" icons next to the field to show only that value or to remove it from the display. We can combine filtering multiple fields in or out to drill down specifically into the logs.

**Fun fact:** Clicking on these filters is actually just applying the relevant KQL syntax.

![image](https://github.com/user-attachments/assets/e8843f42-f7aa-467a-8abc-263383f2d409)

To remove applied filters, simply click on the "**x**" alongside the filter, just below the search bar.

![image](https://github.com/user-attachments/assets/74916624-f2c8-47d7-b5d0-c5718422b8ea)


3. Let's look at the activity of the IP address **10.9.98.230**. We can click on the "clientip" field to see the IPs with the most values.

4. Using the timeline at the top, we can see a lot of activity from this IP address took place between 11:30:00 and 11:35:00. This would be a good place to begin our analysis. Narrow the start date and time to "**Oct 1, 2024 @ 11:30:00.000**" and end date and time to "**Oct 1, 2024 @ 12:00:00.000**".

5. Each log can be expanded by using the "**>**" icon located on the left of the log/document. Fortunately, the logs are pretty small in this instance, so we can browse through them to look for anything untoward.

![image](https://github.com/user-attachments/assets/9980898c-8ebf-4463-97df-b805c5b9e857)

6. After some digging, a few logs stand out. Looking at the **request** field, we can see that a file named "shell.php" has been accessed, with a few parameters "**c**" and "**d**" containing commands. These are likely to be commands input into some form of web shell.

![image](https://github.com/user-attachments/assets/45c5ed4e-9904-4ad6-96bf-ddb60f5df185)

7. Now that we have an initial lead, let’s use a search query to find all logs that contain "**shell.php**". Using the search bar at the top, the query `message: "shell.php"` will search for all entries of "**shell.php**" in the message field of the logs.

![image](https://github.com/user-attachments/assets/a80132b8-0bc5-49a2-aeb4-340fd6166a23)


## 🔴 OPERATION RED 🔴
In this section we will now take a look at the _red_ aspect. In other words, the attack itself and how it was carried out.

### Why Do Websites Allow File Uploads?
File uploads are everywhere on websites, and for good reason. Users often need to upload files like profile pictures, invoices, or other documents to update their accounts, send receipts, or submit claims. These features make the user experience smoother and more efficient. But while this is convenient, it also creates a risk if file uploads aren't handled properly. If not properly secured, this feature can open up various vulnerabilities attackers can exploit.

### File Upload Vulnerabilities
File upload vulnerabilities occur when a website doesn't properly handle the files that users upload. If the site doesn't check what kind of file is being uploaded, how big it is, or what it contains, it opens the door to all sorts of attacks. For example:
- **RCE**: Uploading a script that the server runs gives the attacker control over it.
- **XSS**: Uploading an HTML file that contains an XSS code which will steal a cookie and send it back to the attacker's server.

These can happen if a site doesn't properly secure its file upload functionality.

### Why Unrestricted File Uploads Are Dangerous

Unrestricted file uploads can be particularly dangerous because they allow an attacker to upload any type of file. If the file's contents aren't properly validated to ensure only specific formats like PNG or JPG are accepted, an attacker could upload a malicious script, such as a PHP file or an executable, that the server might process and run. This can lead to code execution on the server, allowing attackers to take over the system.

Examples of abuse through unrestricted file uploads include:
- Uploading a script that the server executes, leading to RCE.
- Uploading a crafted image file that triggers a vulnerability when processed by the server.
- Uploading a web shell and browsing to it directly using a browser.

### Usage of Weak Credentials

One of the easiest ways for attackers to break into systems is through weak or default credentials. This can be an open door for attackers to gain unauthorised access. Default credentials are often found in systems where administrators fail to change initial login details provided during setup. For attackers, trying a few common usernames and passwords can lead to easy access.
Below are some examples of weak/default credentials that attackers might try:

![image](https://github.com/user-attachments/assets/e85d9ce7-0744-41ab-a3a5-cd0de8914c9e)

Attackers can use tools or try these common credentials manually, which is often all it takes to break into the system.

### What is Remote Code Execution (RCE)?
**Remote code execution (RCE)** happens when an attacker finds a way to run their own code on a system. This is a highly dangerous vulnerability because it can allow the attacker to take control of the system, exfiltrate sensitive data, or compromise other connected systems.

### What Is a Web Shell?
A **web shell** is a script that attackers upload to a vulnerable server, giving them remote control over it. Once a web shell is in place, attackers can run commands, manipulate files, and essentially use the compromised server as their own. They can even use it to launch attacks on other systems. 
For example, attackers could use a web shell to:
- Execute commands on the server
- Move laterally within the network
- Download sensitive data or pivot to other services

A web shell typically gives the attacker a web-based interface to run commands. Still, in some cases, attackers may use a reverse shell to establish a direct connection back to their system, allowing them to control the compromised machine remotely. Once an attacker has this level of access, they might attempt privilege escalation to gain even more control, such as achieving root access or moving deeper into the network.

### Exploiting RCE via File Upload

Once an RCE vulnerability has been identified that can be exploited via file upload, we now need to create a malicious file that will allow remote code execution when uploaded. A malicious PHP file could be uploaded to exploit this vulnerability.

### Making the Most of It

Once the vulnerability has been exploited and you now have access to the operating system via a web shell, there are many next steps you could take depending on:
- **a)** what your goal is and
- **b)** what misconfigurations are present on the system, which will determine exactly what we can do. Once you've gained access you could run Linux commands, e.g.: 

![image](https://github.com/user-attachments/assets/23a37b3e-992a-46f6-a90e-f67aa1ab6501)
(etc etc)

### Practical

Your task today is two-fold. First, you must access Kibana on `10.10.214.206:5601` to investigate the attack and answer the **blue** questions below. Then, you will proceed to Frosty Pines Resort's website at `http://frostypines.thm` and recreate the attack to answer the **red** questions and inform the developers what element of the website was vulnerable.

**Note:**
- To review the logs of the attack on Frosty Pines Resorts, make sure you select the "**frostypines-resorts**" collection within ELK.
- The date and time that you will need to use when reviewing logs will be **between 11:30 and 12:00 on October 3rd 2024.**
- To access `http://frostypines.thm` you will need to reference it within your hosts file. On the AttackBox, this can be done by executing the following command in a terminal: `echo "10.10.214.206 frostypines.thm" >> /etc/hosts`

### Answers
1. **BLUE**: Where was the web shell uploaded to? **/media/images/rooms/shell.php**

![image](https://github.com/user-attachments/assets/99dca956-396c-4566-bf13-028caff30208)

- Type `message: "shell.php"` to search for all entries of "**shell.php**" in the message field of the logs.

2. **BLUE**: What IP address accessed the web shell? **10.11.83.34**

![image](https://github.com/user-attachments/assets/d2002f6e-35a4-49c6-87c6-13bb52f62a17)

- There are 2 IP addresses in the `clientip` filter. One isn't executing any commands (no commands shown after`/shell.php`), so nothing malicious there.

![image](https://github.com/user-attachments/assets/30e47dcb-dbdf-4c95-a991-945c45018dcb)

- The other IP address *is* executing commands (can see `/shell.php?command=ls`, `/shell.php?command=id`...). Suggests this IP is the malicious one.

![image](https://github.com/user-attachments/assets/5e9a3f1c-ebc9-43fa-ad55-0a5d55b0bc96)


3. **RED**: What is the contents of the flag.txt? **THM{Gl1tch_Was_H3r3}**
- Execute `echo "10.10.214.206 frostypines.thm" >> /etc/hosts` within a terminal.

![image](https://github.com/user-attachments/assets/1e37fbb0-904a-4145-9529-95bba66e3c14)

- Go to`http://frostypines.thm`

![image](https://github.com/user-attachments/assets/ed47b5ac-2672-4928-8ba8-15f210e36997)

- Need to upload a malicious file somewhere. Can try out some of the poor credentials from before to log in.
- Click Admin (top right corner) -> Admin -> Add new Room (left panel). There's a place files can be uploaded. Change a few fields, e.g. room number, room name, etc.
- Copy and paste text from THM example PHP file into Sublime Text and save file as **shell.php**.
- Browse and find shell.php. **Note:** Make sure "All Files" is selected, not "All Supported Types". Otherwise shell.php file won't be shown.

![image](https://github.com/user-attachments/assets/6d280d5b-3160-4132-848d-2017c512a845)

- Click Add Room, and:

![image](https://github.com/user-attachments/assets/c4172405-c7fa-4668-8645-1e1e8a23c8cc)

- Now to find out where web shell script is being stored. Go to menu -> More Tools -> Web Developer Tools -> Network tab.
- Refresh page.
- Scroll down the list and find the shell.php file. Select it to get more info, including the path where the file is being stored.

![image](https://github.com/user-attachments/assets/129818ee-2428-476a-881d-e5b0d756a0f1)

- Copy paste the path (plus domain name) `http://frostypines.thm/media/images/rooms/shell.php` into browser and...

![image](https://github.com/user-attachments/assets/d3209891-7fed-422f-9a18-31a68fbffa49)

The web shell is working! There is the malicious code in action. Now, we can run commands using this bar, and the output will be displayed. For example, running the command `pwd` now returns:

![image](https://github.com/user-attachments/assets/61db01b2-ac63-4276-8175-932ac0916a71)

- Enter `ls` to list files in current directory.
- Enter `cat flag.txt`.
- The contents of `flag.txt` are: **THM{Gl1tch_Was_H3r3}**.
