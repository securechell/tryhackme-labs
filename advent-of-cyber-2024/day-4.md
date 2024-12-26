# Day 4: I’m all atomic inside!

**Learning Objectives**
- Learn how to identify malicious techniques using the MITRE ATT&CK framework.
- Learn about how to use Atomic Red Team tests to conduct attack simulations.
- Understand how to create alerting and detection rules from the attack tests.

### Detection Gaps
While it might be the utopian dream of every blue teamer, we will rarely be able to detect every attack or step in an attack kill chain. There are gaps in detection. But worry not! There are things we can do to help make these gaps smaller.

Detection gaps are usually for one of two main reasons:

- **Security is a cat-and-mouse game.** As we detect more, the threat actors and red teamers will find new sneaky ways to thwart our detection. We then need to study these novel techniques and update our signature and alert rules to detect these new techniques.
- **The line between anomalous and expected behaviour is often very fine and sometimes even has significant overlap.** For example, let's say we are a company based in the US. We expect to see almost all of our logins come from IP addresses in the US. One day, we get a login event from an IP in the EU, which would be an anomaly. However, it could also be our CEO travelling for business. This is an example where normal and malicious behaviour intertwine, making it hard to create accurate detection rules that would not have too much noise.

Blue teams constantly refine and improve their detection rules to close the gaps they experience due to the two reasons mentioned above. Let's take a look at how this can be done!

### Cyber Attacks and the Kill Chain

All cyber attacks follow a fairly standard process, which is by the Unified Cyber Kill chain. As a blue teamer, it would be our dream to prevent all attacks at the start of the kill chain. But this is not possible. The goal is, therefore, to ensure we can detect the threat actor before the very last phase of goal execution.

### MITRE ATT&CK

A popular framework for understanding the different techniques and tactics that threat actors perform through the kill chain is the MITRE ATT&CK framework. The framework is a collection of tactics, techniques, and procedures (TTPs) that have been seen to be implemented by real threat actors.

However, the framework primarily discusses these TTPs in a theoretical manner. Even if we know we have a gap for a specific TTP, we don't really know how to test the gap or close it down. This is where the Atomics come in!

### Atomic Red
The **Atomic Red Team** library is a collection of red team test cases that are mapped to the MITRE ATT&CK framework. The library consists of simple test cases that can be executed by any blue team to test for detection gaps and help close them down. The library also supports automation, where the techniques can be automatically executed. However, it is also possible to execute them manually.

### Running an Atomic

McSkidy suspects that the supposed attacker used the MITRE ATT&CK technique [T1566.001 Spearphishing](https://attack.mitre.org/techniques/T1566/001/) with an attachment. Let's recreate the attack emulation performed by the supposed attacker and then look for the artefacts created.

1. Open up a PowerShell prompt as administrator. Enter the command `Get-Help Invoke-AtomicTest`.
![image](https://github.com/user-attachments/assets/ecf81f87-c8da-49c1-b8e1-73cf1356d457)

2. **Our First Command**
We would like to know more about what exactly happens when we test the Technique T1566.001. To get this information, we must include the name of the technique we want information about and then add the flag `-ShowDetails` to our command.

Let's construct the command: `Invoke-AtomicTest T1566.001 -ShowDetails`.
- `T1566.001`: The technique you want to emulate.
- `-ShowDetails`: Displays the details of each test included in the Atomic.
![image](https://github.com/user-attachments/assets/d8cb2e4e-d01d-4f82-bea7-5b84eda19a7b)

The output above is clearly split up into multiple parts, each matching a test.

3. **Phishing: Spearphishing Attachment T1566.001 Emulated**
Let's continue and run the first test of T1566.001. Before running the emulation, we should ensure that all required resources are in place. To verify this, we can add the flag `-Checkprereq` to our command.
- Try it with Test 2. The command should look something like this: `Invoke-AtomicTest T1566.001 -TestNumbers 2 -CheckPrereq`.
	- `-TestNumbers`: Sets the tests you want to execute using the test number.
	- `-Checkprereq`: Provides a check if all necessary components are present for testing
![image](https://github.com/user-attachments/assets/7f646bf1-fd4c-403d-aefd-1f7aea2473a5)

Running the command for test 2 states that Microsoft Word needs to be installed.

Now that we have verified the dependencies, let us continue with the emulation.

- Execute the following command to start the emulation: `Invoke-AtomicTest T1566.001 -TestNumbers 1`.
![image](https://github.com/user-attachments/assets/d78276d7-7019-4dd0-b39c-c059745454df)

Based on the output, we can determine that the test was successfully executed. We can now analyse the logs in the Windows Event Viewer to find Indicators of Attack and Compromise.

### Detecting the Atomic

Now that we have executed the T1566.001 Atomic, we can look for log entries that point us to this emulated attack. For this purpose, we will use the Windows Event Logs. This machine comes with [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) installed. System Monitor (Sysmon) provides us with detailed information about process creation, network connections, and changes to file creation time.

1. To make it easier for us to pick up the events created for this emulation, we will first start with cleaning up files from the previous test by running the command `Invoke-AtomicTest T1566.001 -TestNumbers 1 -Cleanup`.
	- `-Cleanup`: Run the cleanup commands that were configured to revert your machine state to normal.

2. Now, we will clear the Sysmon event log:
	- Open up Event Viewer.
	- Navigate to: Applications and Services -> Microsoft -> Windows -> Sysmon -> Operational.
	- Right-click **Operational** on the left-hand side of the screen and select **Clear Log**. Click **Clear** when the popup shows.

3. Now that we have cleaned up the files and the sysmon logs, let us run the emulation again by issuing the command `Invoke-AtomicTest T1566.001 -TestNumbers 1`.
![image](https://github.com/user-attachments/assets/45213faf-21de-46b4-8e2d-ebdf23c25d45)

4. Next, go to the Event Viewer and right-click on the **Operational** log on the left-hand side of the screen, then click on **Refresh**. There should be new events related to the emulated attack.

5. Now sort the table on the Date and Time column to order the events chronologically (oldest first). The first two events of the list are tests that Atomic executes for every emulation. We are interested in **2 events** that detail the attack:
	- First, a process was created for PowerShell to execute the following command: `"powershell.exe" & {$url = 'http://localhost/PhishingAttachment.xlsm' Invoke-WebRequest -Uri $url -OutFile $env:TEMP\PhishingAttachment.xlsm}`.
	- Then, a file was created with the name PhishingAttachment.xlsm.

6. Click on each event to see the details. When you select an event, you should see a detailed overview of all the data collected for that event. Click on the **Details** tab to show all the **EventData**. Let us take a look at the details of these events below. The data highlighted is valuable for incident response and creating alerting rules.
![image](https://github.com/user-attachments/assets/2d28f985-14ae-474f-8bd5-10e7cbfc7b35)

7. Navigate to the directory `C:\Users\Administrator\AppData\Local\Temp\`, and open the file `PhishingAttachment.txt`. (The flag included is the answer to question 1). Make sure to answer the question now, as the cleanup command will delete this file.

8. Let's clean up the artefacts from our spearphishing emulation. Enter the command `Invoke-AtomicTest T1566.001-1 -cleanup`.
![image](https://github.com/user-attachments/assets/65771dc5-1f93-47fc-888a-141e408d2d21)

Now that we know which artefacts were created during this spearphishing emulation, we can use them to create custom alerting rules. In the next section, we will explore this topic further.

### Alerting on the Atomic

In the previous paragraph, we found multiple indicators of compromise through the Sysmon event log. We can use this information to create detection rules to include in our EDR, SIEM, IDS, etc. These tools offer functionalities that allow us to import custom detection rules. There are several detection rule formats, including Yara, Sigma, Snort, and more. Let's look at how we can implement the artefacts related to T1566.001 to create a custom Sigma rule.

Two events contained possible indicators of compromise. Let's focus on the event that contained the `Invoke-WebRequest` command line:

`"powershell.exe" & {$url = 'http://localhost/PhishingAttachment.xlsm' Invoke-WebRequest -Uri $url -OutFile $env:TEMP\PhishingAttachment.xlsm}"`

We can use multiple parts of this artefact to include in our custom Sigma rule.
- `Invoke-WebRequest`: It is not common for this command to run from a script behind the scenes.
- `$url = 'http://localhost/PhishingAttachment.xlsm'`: Attackers often use a specific malicious domain to host their payloads. Including the malicious URL in the Sigma rule could help us detect that specific URL.
- `PhishingAttachment.xlsm`: This is the malicious payload downloaded and saved on our system. We can include its name in the Sigma rule as well.

Combining all these pieces of information in a Sigma rule would look something like this:
![image](https://github.com/user-attachments/assets/32b148e7-d1bc-4c30-9439-e9c79af45277)

The `detection` part is where the effective detection is happening. We can see clearly the artefacts that we discovered during the emulation test. We can then import this rule into the main tools we use for alerts, such as the EDR, SIEM, XDR, and many more.

Now that Glitch has shown us his intentions, let's continue with his work and run an emulation for ransomware.

### Challenge

As Glitch continues to prepare for SOC-mas, he decides to conduct an attack simulation that would mimic a ransomware attack across the environment. He is unsure of the correct detection metrics to implement for this test and asks you for help. Your task is to identify the correct atomic test to run that will take advantage of **a command and scripting interpreter**, conduct the test, and extract valuable artefacts that would be used to craft a detection rule.

### Answers
1. What was the flag found in the .txt file that is found in the same directory as the PhishingAttachment.xslm artefact? **THM{GlitchTestingForSpearphishing}**.
2. What ATT&CK technique ID would be our point of interest? **T1059**
![image](https://github.com/user-attachments/assets/ac7534a9-c120-40fa-9670-ef45c665daf1)

3. What ATT&CK subtechnique ID focuses on the Windows Command Shell? **T1059.003**
4. What is the name of the Atomic Test to be simulated? **Simulate BlackByte Ransomware Print Bombing**.
![image](https://github.com/user-attachments/assets/5a38a97e-83b8-471d-922f-e86c4503d0c6)

5. What is the name of the file used in the test? **Wareville_Ransomware.txt**.
6. What is the flag found from this Atomic Test? **THM{R2xpdGNoIGlzIG5vdCB0aGUgZW5lbXk=}**
	- In PowerShell type `Invoke-AtomicTest T1059.003 -TestNumbers 4` (because that BlackByte Ransomware test was Atomic Test Number **4**).
	- Save the PDF.
	- Read the contents:
![image](https://github.com/user-attachments/assets/252e9d70-c49d-44fd-aaad-8568ec708b30)
