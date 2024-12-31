# Setting up a SOC Lab (Splunk)
### **Task 1: Intro**
In this room, you will be handed over two VMs, Linux and Windows, and your task will be to install Splunk on both machines.  

**Learning Objectives**
- Dive deep into the Splunk installation process.
- How to install and configure Splunk in Linux and Windows Environments.  
- How to integrate different log sources into Splunk.

### **Task 2: Setting up a Lab**
Splunk is a SIEM solution. This room will cover installing Splunk on Linux/Windows. Each lab covers the following topics:

**Linux Lab**
- Install Splunk on Ubuntu Server
- Install and integrate Universal Forwarder
- Collecting Logs from important logs sources/files like syslog, auth.log, audited, etc

**Windows Lab**
- Install Splunk on Windows Machine
- Install and Integrate the Universal Forwarder
- Integrating and monitoring Coffely. THM’s weblogs
- Integrating Windows Event Logs

### **Task 3: Deployment on Linux Server**
In terminal:
- `cd Downloads/splunk/` to find the Splunk installer
- `ls`
- `sudo su` to change to the root user before applying commands (will go from `ubuntu@coffely:` to `root@coffely:`)
- `tar xvzf splunk_installer.tgz` to install and unzip Splunk
- `ls` when installation is complete to see the newly created folder called "Splunk"
- `mv splunk /opt/` to move `splunk` folder to the `/opt/` directory
- `cd /opt/splunk/bin/`
-  run the command `./splunk start --accept-license` to start Splunk
  
![THM - Setting up a SOC Lab (Splunk)](https://github.com/user-attachments/assets/441beeed-d595-479a-b6e2-fbeddd7d67d4)
- Create admin username and password
- Congrats! - We successfully installed Splunk on our Linux machine
- go to `http://coffely:8000` in VM browser to access Splunk

**Answers**:
1. What is the default port for Splunk? **8000** (given away by: `....coffely:8000`)

### **Task 4: Interacting with CLI**
Time to learn some key commands while interacting with Splunk instances through CLI. These commands are run from the `/opt/splunk/` directory as root. If not already root, type in: `sudo su`, then `cd /opt/splunk`
![THM - Setting up a SOC Lab (Splunk)-1](https://github.com/user-attachments/assets/c3cd124a-afd0-41de-83d3-2d145d4a2d22)

Some important commands are:
- `splunk start` to start the Splunk server
![THM - Setting up a SOC Lab (Splunk)-2](https://github.com/user-attachments/assets/0f4e6c2c-a57a-4ad1-bc86-c4fa06e380d4)

- `splunk stop` stops all the running Splunk processes and disables the server from accepting incoming data
![THM - Setting up a SOC Lab (Splunk)-3](https://github.com/user-attachments/assets/5c09022a-9c71-43df-b2de-fb3695dd123d)

- `splunk restart` stops all running Splunk processes and then starts them again
- `splunk status` displays information about the current state of the server, including whether it is running or not, and any errors that may be occurring
![THM - Setting up a SOC Lab (Splunk)-4-1](https://github.com/user-attachments/assets/9441eaf3-96d4-4c2a-9b79-1418d29b1606)

- `splunk add oneshot` used to add a single event to the Splunk index. This is useful for testing purposes or for adding individual events that may not be part of a larger data stream
-  `splunk search` used to search for data in the Splunk index
![THM - Setting up a SOC Lab (Splunk)-5](https://github.com/user-attachments/assets/45312354-3d46-46b2-939a-9f71cc978577)

- `splunk help` provides all the help options; the most important command!

**Answers**: 
1. In Splunk, what is the command to search for the term coffely in the logs? `./bin/splunk search coffely`

### **Task 5: Data Ingestion**
Configuring data ingestion is an important part of Splunk. This allows for the data to be indexed and searchable for the analysts. Splunk accepts data from various log sources like Operating System logs, Web Applications, Intrusion Detection logs, etc. In this task, we will use **Splunk Forwarder** to ingest the Linux logs into our Splunk instance. In this task, we will be installing and configuring Universal Forwarders.

- `cd Downloads/splunk/` to get to where the forwarder is
- `sudo su` install must be done as root
- `tar xvzf splunkforwarder.tgz` to install all the files in the `splunkforwarder` folder
- `mv splunkforwarder /opt/`
- `cd /opt/splunkforwarder`
- `./bin/splunk start --accept-license`
- Create admin username and password
- Received an error message:
![THM - Setting up a SOC Lab (Splunk)-6](https://github.com/user-attachments/assets/0c3e661d-991a-4670-9d83-15d59f2cd76a)

- By default, Splunk forwarder runs on port 8089, but that port was unavailable so I used port 8090.

**Answers**:
1. What is the default port, which Splunk Forwarder runs on? **8089**

### **Task 6: Configuring Forwarder on Linux**
Now that we have installed the forwarder, it needs to know where to send the data. So we will configure it on the host end to send the data and configure Splunk so that it knows from where it is receiving the data.

**Splunk Configuration**
- Log into Splunk
- Settings -> Forwarding and receiving
- Click Configure receiving
- Click New Receiving Port, enter 9997 and Save. By default, the Splunk instance receives data from the forwarder on the port `9997`. That's why we're configuring our Splunk to start **listening on port 9997**

**Creating Index**
Now to create an index that will store all the receiving data. If we do not specify an index, it will start storing received data in the default index, which is called the `main` index.
- Settings -> Indexes. This tab contains all the indexes created by the user or by default
- Click New Index, create an index with name `Linux_host`. Save.

**Configuring Forwarder**
In Linux host terminal:
- go to `/opt/splunkforwarder/bin` directory
- type `./splunk add forward-server 10.10.202.206:9997` to add the forwarder server, which listens to port 9997
![THM - Setting up a SOC Lab (Splunk)-7](https://github.com/user-attachments/assets/9b763016-c539-4ab2-b19f-7d30123b7f51)

**Linux Log Sources**
Linux stores all its important logs into the `/var/log` file. In our case, we will ingest **syslog** into Splunk. Other logs can be ingested using the same method.
- type `./splunk add monitor /var/log/syslog -index Linux_host` to tell Splunk Forwarder to monitor the `/var/log/syslog` file
![THM - Setting up a SOC Lab (Splunk)-8](https://github.com/user-attachments/assets/64dbcbdb-e84b-4a61-bb7b-b5b1973883c2)

**Exploring inputs.conf**
- `cd /opt/splunkforwarder/etc/apps/search/local` to go to where the  **inputs.conf** file is located
- then `cat inputs.conf`  to see the contents of the file
![THM - Setting up a SOC Lab (Splunk)-9](https://github.com/user-attachments/assets/33fa54ca-3510-439c-90f8-9d6fcf3395ee)

**Utilising Logger Utility**
Logger is a built-in command line tool to create test logs added to the syslog file. As we are already monitoring the syslog file and sending all logs to the Splunk, the log we generate in the next step can be found with Splunk logs. To run the command, use the following command.
- (first `cd /opt/splunkforwarder/bin`)
- `logger "coffely-has-the-best-coffee-in-town"`
- `tail -1 /var/log/syslog`
![THM - Setting up a SOC Lab (Splunk)-10](https://github.com/user-attachments/assets/9fcc626a-62ee-4e97-912b-795fe4ca2ed0)

- See Splunk events; the Logger input will be there:
![THM - Setting up a SOC Lab (Splunk)-11](https://github.com/user-attachments/assets/57e41310-9de8-498c-9d21-d3f0b79de481)

**Answers**:
1. Follow the same steps and ingest `/var/log/auth.log` file into Splunk index Linux_logs. What is the value in the sourcetype field? **syslog**
	- Ingest `/var/log/auth.log` file into Splunk index
![THM - Setting up a SOC Lab (Splunk)-12](https://github.com/user-attachments/assets/fd0c282a-286e-4887-96bf-cc1c3b9eb70e)
	- Check Splunk
![THM - Setting up a SOC Lab (Splunk)-13](https://github.com/user-attachments/assets/ce2f8464-b2dc-433a-9675-7c88af8cb6aa)

2. Create a new user named analyst using the command `adduser analyst`. Once created, look at the events generated in Splunk related to the user creation activity. How many events are returned as a result of user creation? **6**
	- type `adduser analyst` in `splunkforwarder` directory to create user
![THM - Setting up a SOC Lab (Splunk)-14](https://github.com/user-attachments/assets/9190556d-0881-474b-81f5-7b3c48c0311f)
	- type "analyst" in Splunk search bar; states there are 6 events
![THM - Setting up a SOC Lab (Splunk)-15](https://github.com/user-attachments/assets/35bd2367-ce68-4e53-8d4c-2a91979637aa)
![THM - Setting up a SOC Lab (Splunk)-16](https://github.com/user-attachments/assets/ee0b9fb1-ad77-4066-8931-1620a4ed6fd3)

3. What is the path of the group the user is added after creation? **/etc/group**
	- See Splunk:
![THM - Setting up a SOC Lab (Splunk)-17](https://github.com/user-attachments/assets/ff170d81-6a28-4a6a-97c5-d76492709e14)

### **Task 7: Installing on Windows**
- To download Splunk: File Explorer -> Downloads folder -> Splunk-Instance
- By default it will install Splunk in the folder C:\Program Files\Splunk

**Answers**:
1. What is the default port Splunk runs on? **8000**
2. Click on the Add Data tab; how many methods are available for data ingestion? **3**
3. Click on the Monitor option; what is the first option shown in the monitoring list? **Local Event Logs**

### **Task 8: Installing and Configuring Forwarder**
- First, we will configure the receiver on Splunk so the forwarder knows where to send the data: Installing Splunk Forwarder-> New Receiving Port -> type "9997" -> Save
- Then install Splunk Forwarder
- Then go to Settings -> Forwarder management. Our host details are there:
<img width="472" alt="THM - Setting up a SOC Lab (Splunk)-18" src="https://github.com/user-attachments/assets/6c789da3-9304-4c2a-9867-dcdb6b72f001" />

**Answers**:
1. What is the full path in the C:\Program Files where Splunk forwarder is installed? **C:\Program Files\SplunkUniversalForwarder** (find it through File Explorer)
2. What is the default port on which Splunk configures the forwarder? **9997** (this was in the Receiving Indexer part of the Forwarder setup process)

### **Task 9: Ingesting Windows Logs**
We have installed the forwarder and set up the listener on Splunk. It's time to configure Splunk to receive Event Logs from this host and configure the forwarder to collect Event Logs from the host and send them to the Splunk Indexer.

**Select Forwarders**
- Settings -> Add data -> Forward (to get the data from Splunk Forwarder)
- Click on the host "coffelylab" in the Available host(s) section; it will be added to the Selected host(s) section.
- In New Server Class Name write "coffely_lab" (for e.g.)
- Next

**Select Source**
It's time to select the log source that we need to ingest. The list shows many log sources to choose from.
- Click Local Event Logs to configure receiving Event Logs from the host. Different Event Logs will appear in the list to choose from
- Click Application & Security & System so they go into the Selected items section
- Next

**Input Settings**
Create an index that will store the incoming Event logs. Once created, select the Index from the list and move to the next step.
- Click Create a new index
- Give the Index name "win_logs" (for e.g.)
- Review

**Review**
<img width="350" alt="THM - Setting up a SOC Lab (Splunk)-19" src="https://github.com/user-attachments/assets/8cf03fc1-23d1-4c36-8f46-00f95730a6b9" />
- Submit

**Done**
Click Start Searching. It will take us to the Search App. If everything goes smoothly, we will receive the Event Logs immediately.

**Answers**:
1. While selecting Local Event Logs to monitor, how many Event Logs are available to select from the list to monitor? **5**. (In Local Event Logs in the Select Source section, there were 5 logs to choose from: Application, ForwardedEvents, Security, Setup, and System)
<img width="458" alt="THM - Setting up a SOC Lab (Splunk)-21" src="https://github.com/user-attachments/assets/8008c02c-b87c-4dee-8c9c-3850987307ba" />

2. Search for the events with EventCode=4624. What is the value of the field Message? **An account was successfully logged on**. (Interesting Fields -> EventCode -> 4624. Then click any of the events and look at the Message)
<img width="396" alt="THM - Setting up a SOC Lab (Splunk)-20" src="https://github.com/user-attachments/assets/149f2fd9-6cad-49ed-9b9a-2651da855e0b" />

### **Task 10: Ingesting Coffely Web Logs**
You are asked to configure Splunk to receive web logs to trace orders and improve coffee sales. Now let's ingest web logs into Splunk. Go to Settings -> Add Data -> Forward.

**Select Forwarder**
- Click WINDOWS coffelylab in the Available host(s) section
- Put the Class Name as "web_logs" (for e.g.)
- Next

**Select Source**
Web logs are placed in the directory C:\inetpub\logs\LogFiles\W3SVC*. (Mine was C:\inetpub\logs\LogFiles\W3SVC***1***)
- Select Files & Directories
- In File or Directory section put: C:\inetpub\logs\LogFiles\W3SVC1
- Next

**Input Settings**
Next, we will select the source type for our logs. As our web is hosted on an IIS server, we will choose this option and create an appropriate index for these logs.
- Select -> Select Source Type
- Type IIS
<img width="392" alt="THM - Setting up a SOC Lab (Splunk)-22" src="https://github.com/user-attachments/assets/72406b24-64c4-4c18-ac19-9d10621487c3" />
- Review

Now everything is done. It's time to see if we get the weblogs in our newly created index. Let's visit the website `coffely.thm` and generate some logs. The logs should start propagating in the search tab.
<img width="454" alt="THM - Setting up a SOC Lab (Splunk)-24" src="https://github.com/user-attachments/assets/8c7f0d9f-7a68-420b-ad2a-b1adfe22b954" />

**Answer**:
1. In the lab, visit `http://coffely.thm/secret-flag.html`; it will display the history logs of the orders made so far. Find the flag in one of the logs.  **{COffely_Is_Best_iN_TOwn}**

### Conclusion
Understanding the process of installation and configuration of any SIEM solution and then ingesting logs from various sources is a very important concept for a SOC analyst. In this room, we learned how to:

- Install Splunk both on Linux and Windows Host.
- Install Splunk Forwarder on Linux and Windows Host.
- Configure Splunk to receive OS-based and Web logs.
