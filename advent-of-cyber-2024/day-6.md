# Day 6: If I can't find a nice malware to use, I'm not going.

**Learning Objectives**
- Analyse malware behaviour using sandbox tools.
- Explore how to use YARA rules to detect malicious patterns.
- Learn about various malware evasion techniques.
- Implement an evasion technique to bypass YARA rule detection.

### Detecting Sandboxes
A **sandbox** is an isolated environment where (malicious) code is executed without affecting anything outside the system. Often, multiple tools are installed to monitor, record, and analyse the code's behaviour.

Mayor Malware knows that before his malware executes, it needs to check if it is running on a Sandbox environment (he hopes it isn't!). If it is, then it should not continue with its malicious activity.

To do so, he has settled on one technique - an "Anti-Sandbox Technique" - which checks if the directory `C:\Program Files` is present by querying the Registry path `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion`. The value can be confirmed by visiting the Registry path within the Registry Editor, as shown below:

![image](https://github.com/user-attachments/assets/4e1abae9-48c3-48e3-826a-92d51f69478e)

- To open the Registry Editor (pic above), navigate to the Start Menu on the bottom -> Windows System -> Run -> type "regedit", and click OK.

This directory is often absent on sandboxes or other virtualised environments, which could indicate that the malware is running in a sandbox.

### Can YARA Do It?
**YARA** is a tool used to identify and classify malware based on patterns in its code. By writing custom rules, analysts can define specific characteristics to look for—such as particular strings, file headers, or behaviours—and YARA will scan files or processes to find matches, making it invaluable for detecting malicious code.

Mayor Malware does not think such a simple tool can detect his malware. But just to be sure, he has to test it out himself.

To do this, he wrote a small script that executes a YARA detection rule every time a new event is added to the System monitor log. This particular YARA rule detects any command that tries to access the registry. Let's have a look at the rule:

![image](https://github.com/user-attachments/assets/4d54d6b3-0feb-4ad8-8cea-5170fbef3bc8)

Let's understand the contents:

- In the **strings** section, we have defined variables that include the value to look out for: $cmd
- In the **condition** section, we define when the rule will match the scanned file. In this case, if any of the specified strings are present. 

For his testing, Mayor Malware has set up a one-function script that runs the Yara rule and **logs a true positive** in `C:\Tools\YaraMatches.txt`.

1. Open up a PowerShell window, navigate to the `C:\Tools` directory, and use the following command to start up the EDR (endpoint detection and response):

![image](https://github.com/user-attachments/assets/8a99bb31-e464-4a5b-9bd8-8f42fed68b15)

![image](https://github.com/user-attachments/assets/c6b3dc99-e1bf-442b-9290-9b4d51654500)

This tool will run on the system and continuously monitor the generated Event Logs. It will alert you if it finds any activity/event that indicates the registry is being queried.

2. Run it again: we see that no events have been detected.

![image](https://github.com/user-attachments/assets/27668d95-e2bd-466c-864d-d1ecf94897cf)

3. Now run the malware by navigating to `C:\Tools\Malware`, and double-clicking on `MerryChristmas.exe`. We can see the event was detected:

![image](https://github.com/user-attachments/assets/d156883e-8907-4675-8675-fc10f789bfbf)

If our custom script did its job, you should have witnessed a popup by our EDR with a flag included, as shown below. This will be the answer to Question 1.

![image](https://github.com/user-attachments/assets/d2b02438-4aa9-42dd-a303-074431220093)

4. You can now exit the custom EDR by pressing Ctrl+C.

### Adding More Evasion Techniques
Ah, it seems that Yara can detect the evasion that Mayor Malware has added. No worries. Because we can make our malware even stealthier by introducing **obfuscation**.

### Beware of Floss
While obfuscation is helpful, we also need to know that there are tools available that extract obfuscated strings from malware binaries. One such tool is **Floss**, a powerful tool developed by Mandiant that functions similarly to the Linux strings tool but is optimized for malware analysis, making it ideal for revealing any concealed details.

1. To try out Floss, open a PowerShell Window and enter the following command: `.\FLOSS\floss.exe C:\Tools\Malware\MerryChristmas.exe |Out-file C:\tools\malstrings.txt`.

- Let's break down the command:
	- `floss.exe C:\Tools\Malware\MerryChristmas.exe`: This command scans for strings in the binary MerryChrismas.exe. If any hardcoded variables were defined in the malware, Floss should find them.
	- The `|` command transfers the output from the command on its *left* (`C:\Tools\Malware\MerryChristmas.exe`) into the input of the command on its *right* (`Out-file C:\tools\malstrings.txt`).
	- `Out-file C:\tools\malstrings.txt`: We save the command results in a file called `malstrings.txt`.

![image](https://github.com/user-attachments/assets/a46d9afb-7fab-4e74-9800-3e38eae29872)

2. Once the command is done, open `malstrings.txt`:

![image](https://github.com/user-attachments/assets/6dda432d-19c0-40bb-8728-af15c4cfd14c)

3. Press CTRL+F and search for "THM{" to find the flag as the answer to question two.

### Using YARA Rules on Sysmon Logs
These YARA rules are becoming a pain to Mayor Malware's backside.

If he wants his malware to be undetectable, he needs to research how YARA rules can be used to stop him. For example, his research tells him that YARA rules can also be used to check Sysmon logs for any artefacts left by malware! He'll need to test this as well.

**Sysmon**, a tool from Microsoft's Sysinternals suite, continuously monitors and logs system activity across reboots. This Windows service provides detailed event data on process creation, network connections, and file changes—valuable insights when tracing malware behaviour.

>**Note:**
>Sysmon refers to System Monitor, which is a Windows system service and device driver developed by Microsoft that is designed to monitor and log various events happening within a Windows system.

A YARA rule will look for events with `event id 1: Process created` for this to work. There are many entries in the Sysmon log. To make it easier to find the event we are looking for, we will apply a custom filter using the `EventRecordID` that we can see in the log `YaraMatches.txt` located in `C:\Tools`.

1. Open a PowerShell window and enter the following command to check the contents of the EDR log file: `get-content C:\Tools\YaraMatches.txt`

![image](https://github.com/user-attachments/assets/552a0145-71b9-4100-8276-0c89952ea188)

Note down the `Event Record ID value`: **128352**. We will use this value to create a custom filter in the Windows Event Viewer.

2. Next, open the Windows Event Viewer by clicking on its logo in the taskbar and, on the left-hand side, navigate to Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational. Continue by navigating to Filter Current Log on the right-hand side of the screen.

3. Navigate to XML and tick the checkbox Edit query manually. Click Yes to confirm. Finally, copy the filter into the input box:

![image](https://github.com/user-attachments/assets/8664c4e2-ca9a-4a86-92c7-b2380155e81c)

4. Replace the `EventRecordID` value with the one you recorded before (128352). Apply the filter by clicking OK. Now you get the event related to the malware. Click on the event and then on the Details tab:

![image](https://github.com/user-attachments/assets/2f69a0f7-7968-44e9-9053-9256a6b069cf)

Let's take a look at the `EventData` that is valuable to us:

- The `ParentImage` key shows us which parent process spawned the cmd.exe process to execute the registry check. We can see it was our malware located at `C:\Tools\Malware\MerryChristmas.exe`.
- The `ParentProcessId` and `ProcessId` keys are valuable for follow-up research. We could also use them to check other logs for related events.
- The `User` key can help us determine which privileges were used to run the `cmd.exe` command. The malware could have created a hidden account and used that to run commands.
- The `CommandLine` key shows which command was run in detail, helping us identify the malware's actions.
- The `UtcTime` key is essential for creating a time frame for the malware's operation. This time frame can help you focus your threat hunting efforts.


### Answers
1. What is the flag displayed in the popup window after the EDR detects the malware? **THM{GlitchWasHere}**.
2. What is the flag found in the malstrings.txt document after running floss.exe, and opening the file in a text editor? **THM{HiddenClue}**.
