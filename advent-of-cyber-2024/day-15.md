# Day 15: Be it ever so heinous, there's no place like Domain Controller.

**Learning Objectives**
- Learn about the structures of Active Directory.
- Learn about common Active Directory attacks.
- Investigate a breach against an Active Directory.

### Introducing Active Directory
Before diving into **Active Directory**, let us understand how network infrastructures can be mapped out and ensure that access to resources is well managed. This is typically done through **Directory Services,** which map and provide access to network resources within an organisation. The **Lightweight Directory Access Protocol (LDAP)** forms the core of Directory Services. It provides a mechanism for accessing and managing directory data to ensure that searching for and retrieving information about subjects and objects such as users, computers, and groups is quick.

Active Directory (AD) is, therefore, a Directory Service at the heart of most enterprise networks that stores information about objects in a network. The associated objects can include: Users, Groups, Computers, Printers and other resources.

Building blocks of an AD architecture include: Domains, Organisational Units (OUs), Forest, and Trust Relationships.

#### Group Policy
One of Active Directory's most powerful features is **Group Policy**, which allows administrators to enforce policies across the domain. Group Policies can be applied to users and computers to enforce password policies, software deployment, firewall settings, and more.

**Group Policy Objects (GPOs)** are the containers that hold these policies. A GPO can be linked to the entire domain, an OU, or a site, giving the flexibility in applying policies.

Let us say that McSkidy wants to ensure that all users within Wareville's SOC follow a strict password policy, enforcing minimum password lengths and complexity rules. Here is how it would be done:

1. Go to Windows Icon -> Windows System -> Run. Using the Run window, open Group Policy Management from your server by typing `gpmc.msc`.
2. Right-click your domain and select **"Create a GPO in this domain, and Link it here"**. Name the new GPO **"Password Policy"**.
![image](https://github.com/user-attachments/assets/a88da900-7d80-41f5-a679-b6be5a357987)

3. Edit the GPO by navigating to **Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Password Policy**.
4. Configure the following settings:
    - Minimum password length: 12 characters
    - Enforce password history: 10 passwords
    - Maximum password age: 90 days
    - Password must meet complexity requirements: Enabled
5. Click **OK**, then link this GPO to the domain or specific OUs you want to target.

This policy will now be applied across the domain, ensuring all users meet these password requirements.

### Common Active Directory Attacks
Adversaries are always looking for ways to breach and exploit Active Directory environments to destabilise and cause havoc to organisations. Working with Glitch to secure SOC-mas requires us to know common attacks and their mitigation measures.

### Investigating an Active Directory Breach
#### Group Policy

As previously discussed in this task, Group Policy is a means to distribute configurations and policies to enrolled devices in the domain. For attackers, Group Policy is a lucrative means of spreading malicious scripts to multiple devices.

Reviewing Group Policy Objects (GPOs) is a great investigation step.

### Event Viewer
Windows comes packaged with the Event Viewer. This invaluable repository stores a record of system activity, including security events, service behaviours, and so forth.

For example, within the "Security" tab of Event Viewer, we can see the history of user logins, attempts and logoffs. All categories of events are given an event ID. The table below provides notable event IDs for today's task.
![image](https://github.com/user-attachments/assets/fd586111-ae08-4d80-996b-605d2fca9b2d)

### User Auditing
User accounts are a valuable and often successful method of attack. You can use Event Viewer IDs to review user events and PowerShell to audit their status. Attack methods such as password spraying will eventually result in user accounts being locked out, depending on the domain controller's lockout policy.

To view all locked accounts, you can use the Search-ADAccount cmdlet, applying some filters to show information such as the last time the user had successfully logged in.

`Search-ADAccount -LockedOut | Select-Object Name, SamAccountName, LockedOut, LastLogonDate, DistinguishedName`

### Reviewing PowerShell History and Logs
PowerShell, like Bash on Linux, keeps a history of the commands inputted into the session. Reviewing these can be a fantastic way to see recent actions taken by the user account on the machine.

On a Windows Server, this history file  is located at `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`.

You can use the in-built Notepad on Windows or your favourite text editor to review the PowerShell command history.

Additionally, logs are recorded for every PowerShell process executed on a system. These logs are located within the Event Viewer under Application and Services Logs -> Microsoft -> Windows -> PowerShell -> Operational, or also under Application and Service Logs -> Windows PowerShell. The logs have a wealth of information useful for incident response.

### Practical
Your task for today is to investigate WareVille's SOC-mas Active Directory controller for the suspected breach. Answer the questions below to confirm the details of the breach.

### Answers
*Use the "Security" tab within Event Viewer to answer questions 1 and 2.*
1. On what day was Glitch_Malware **last logged in**? Answer format: DD/MM/YYYY.
	- Go to Event Viewer -> Windows Logs -> Security tab.
	- On right-hand side, under Actions click Find.
	- Enter "Glitch_Malware" in textbox. Click Find Next, then Cancel.
![image](https://github.com/user-attachments/assets/4c314403-ef1b-4109-a3d9-222873441e3d)

We can see the events associated with the username Glitch_Malware:
![image](https://github.com/user-attachments/assets/56813f7e-7b6a-4c57-b87a-d43c8ccebdf9)

![image](https://github.com/user-attachments/assets/34a8ca94-ab62-42a3-a89c-9fec6385a499)

We need to find when he last logged in.
- Go to Find -> Find Next again. Run through until you see Logon in the Task Category.
- Seems to be "11/7/2024".
![image](https://github.com/user-attachments/assets/8296566d-6a0d-4a40-94ee-f3704b6942da)

This is the American MM/D/YYYY format. As DD/DM/YYYY the answer is:  **07/11/2024**.

2. What event ID shows the login of the Glitch_Malware user? **4624**.
![image](https://github.com/user-attachments/assets/0cd01184-97a2-46f6-aec7-15c6a5d0a532)

(Don't go with Special Logon in Task Category, look for a regular Logon)

3. Read the PowerShell history of the Administrator account. What was the command that was used to enumerate Active Directory users?
	- Go to File Explorer -> This PC -> Local Disk (C:) -> Users -> Administrator.
	- Click View (menu bar) and tick Hidden Items.
	- From Administrator got to App Data -> Roaming -> Microsoft -> Windows -> PowerShell -> PSReadLine -> ConsoleHost_history
![image](https://github.com/user-attachments/assets/0642f11b-6947-4ae7-94fd-1b9d2bf832de)

	- Answer: **Get-ADUser -Filter * -Properties MemberOf | Select-Object Name**

4. Look in the PowerShell log file located in Application and Services Logs -> Windows PowerShell. What was Glitch_Malware's set password?
	- Go to Event Viewer.
	- In Windows PowerShell go to Find and look for "password".
	- Run through until you find an event associated with Glitch_Malware
![image](https://github.com/user-attachments/assets/0e0fad99-744b-496b-91e0-5e4905b8dbc6)

	- Answer: **SuperSecretP@ssw0rd!**

5. Review the Group Policy Objects present on the machine. What is the name of the installed GPO?
	- Go to PowerShell
	- Enter `Get-GPO -All`. Scroll down looking for anything suspicious!
![image](https://github.com/user-attachments/assets/2808bafb-73cb-4ae9-8e4c-7decaf489208)

	- Answer: **Malicious GPO - Glitch_Malware Persistence**.
