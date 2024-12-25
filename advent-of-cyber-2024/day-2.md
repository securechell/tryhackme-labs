# Day 2: One man's false positive is another man's potpourri.

### True Positives or False Positives?
In a SOC, events from different devices are sent to the SIEM, which is the single source of truth where all the information and events are aggregated. Certain rules are defined to identify malicious or suspicious activity from these events. If an event or set of events fulfils the conditions of a rule, it triggers an alert. A SOC analyst then analyses the alert to identify if the alert is a **True Positive (TP)** or a **False Positive (FP)**. An alert is considered a TP if it contains actual malicious activity. On the flip side, if the alert triggers because of an activity that is not actually malicious, it is considered an FP. This might seem very simple in theory, but practically, separating TPs from FPs can be a tedious job. It can sometimes become very confusing to differentiate between an attacker and a system administrator.

### Making a Decision
While it is confusing to differentiate between TPs and FPs, it is very crucial to get it right. If a TP is falsely classified as an FP, it can lead to a significant impact from a missed cyber attack. If an FP is falsely classified as a TP, precious time will be spent focusing on the FP, which might lead to less focus on an actual attack. So, how exactly do we ensure that we perform this crucial job effectively? We can use the below pointers to guide us.

#### Using the SOC Superpower
The SOC has a superpower. When they are unsure whether an activity is performed by a malicious actor or a legitimate user, they can just confirm with the user. This privilege is not available to the attacker. A SOC analyst, on the other hand, can just send an email or call the relevant person to get confirmation of a certain activity. In mature organisations, any changes that might trigger an alert in the SOC often require Change Requests to be created and approved through the IT change management process. Depending on the process, the SOC team can ask the users to share Change Request details for confirmation. Surely, if it is a legitimate and approved activity, it must have an approved Change Request.

#### Context
While it might seem like using the SOC superpower makes things super easy, that is not always the case. There are cases which can act as Kryptonite to the SOC superpower:
- If an organisation doesn't have a change request process in place.
- The performed activity was outside the scope of the change request or was different from that of the approved change request.
- The activity triggered an alert, such as copying files to a certain location, uploading a file to some website, or a failed login to a system. 
- An insider threat performed an activity they are not authorised to perform, whether intentionally or unintentionally.
- A user performed a malicious activity via social engineering from a threat actor.

In such scenarios, it is very important for the SOC analyst to understand the context of the activity and make a judgement call based on their analysis skills and security knowledge. While doing so, the analyst can look at the past behaviour of the user or the prevalence of a certain event or artefact throughout the organisation or a certain department. For example, if a certain user from the network team is using Wireshark, there is a chance that other users from the same team also use Wireshark. However, Wireshark seen on a machine belonging to someone from HR or finance should rightfully raise some eyebrows.

#### Correlation

When building the context, the analyst must correlate different events to make a story or a timeline. Correlation entails using the past and future events to recreate a timeline of events. When performing correlation, it is important to note down certain important artefacts that can then be used to connect the dots. These important artefacts can include IP addresses, machine names, user names, hashes, file paths, etc.

Correlation requires a lot of hypothesis creation and ensuring that the evidence supports that hypothesis. A hypothesis can be something like the user downloaded malware from a spoofed domain. The evidence to support this can be proxy logs that support the hypothesis that a website was visited, the website used a spoofed domain name, and a certain file was downloaded from that website.

### Is this a TP or an FP?
Similar to every SOC, the analysts in the Wareville SOC also need to differentiate TPs from FPs. This becomes especially difficult for them near Christmas when the analysts face alert fatigue. To make matters worse, the office of the Mayor has sent the analysts an alert informing them of multiple encoded PowerShell commands run on their systems. Perhaps we can help with that.

1. Connect to the Elastic SIEM.
2. Once we log in, we can click the **hamburger menu** in the top-left corner -> **Discover** to see the events.
3. According to the alert sent by the Mayor's office, the activity occurred on **Dec 1st 2024**, between **0900 and 0930**. We can set this as our time window by clicking the timeframe settings in the upper-right corner. (*Note*: click the **Absolute** tab and set the exact timeframe we want to view). Click the **Update** button to apply the changes.
![image](https://github.com/user-attachments/assets/8f013e56-1e76-4981-a175-c3a39800b577)

4. After updating the settings, we see 21 events in the mentioned timeframe. In their current form, these events don't look very easily readable.
![image](https://github.com/user-attachments/assets/d8c2b3b6-4b53-42d7-a828-c1875dfff95e)

We can use the fields in the left pane to add columns to the results and make them more readable. Hovering on the field name in the left pane will allow adding that field as a column, as shown below.
![image](https://github.com/user-attachments/assets/a899961c-9c01-4f6a-90a8-a35382d6f5da)

5. Since we are looking for events related to PowerShell, we would like to know the following details about the logs.
	- The hostname where the command was run. We can use the `host.hostname` field as a column for that.
	- The user who performed the activity. We can add the `user.name` field as a column for this information.
	- We will add the `event.category` field to ensure we are looking at the correct event category.
	- To know the actual commands run using PowerShell, we can add the `process.command_line` field.
	- Finally, to know if the activity succeeded, we will add the `event.outcome` field.

Once we have added these fields as columns, we will see the results in a format like this.
![image](https://github.com/user-attachments/assets/a57131fa-f023-4da4-8a40-54d4ec32eaf8)

Interesting! So, it looks like someone ran the same encoded PowerShell command on multiple machines. Another thing to note here is that before each execution of the PowerShell command (`process.command_line`), we see an authentication event (`event.category`), which was successful (`event.outcome`).
![image](https://github.com/user-attachments/assets/bbc9bba9-6314-4d13-8610-f616102f6bad)

This activity is observed individually on each machine (`host.hostname` field), and the time difference between the login and PowerShell commands looks very precise (11 secs). Best practices dictate that named accounts are used for any kind of administrator activity so that there is accountability and attribution for each administrative activity performed.

The usage of a generic admin account here also seems suspicious. On asking, the analysts informed us that this account is used by two administrators who were not in the office when this activity occurred. *Hmmm, something is definitely not right. Are these some of Glitch's shenanigans? Is Christmas in danger? We need to find out who ran these commands.*

6. Let's also add the `source.ip` field as a column to find out who ran the PowerShell commands.
![image](https://github.com/user-attachments/assets/90cb2fc5-d24a-4d0a-9396-6f69b5bde368)

Since the `source.ip` field is only available for the authentication events, we can filter out the process events to see if there is a pattern. To do that, we can hover over the `event.category` field in one of the process events. We will see the option to filter only for this value (+ sign) or filter out the value (- sign), as seen below.

![image](https://github.com/user-attachments/assets/fe50ffd6-6edd-4959-b3d1-32060113d24f)

Let's filter for authentication events by clicking the plus (+) sign beside it to show only those in the results.
![image](https://github.com/user-attachments/assets/2a417708-54a7-40fa-93ab-af3df83bf17b)

As a result, you can see that the output only renders the authentication events. Since the result does not give useful insights, **let's remove it for now**. You can do this by clicking the "x" beside the filter.

7. Since the timeframe we previously used was for the PowerShell events, and the authentication events might have been coming from *before* that, we will need to expand the search to understand the context and the historical events for this user. Let's see if we have any events from the user from the 29th of November to the 1st of December. Updating the time filter for these days, the results look like this.
![image](https://github.com/user-attachments/assets/e19e7350-1220-4d41-91b0-9a425274b75b)

Woah, there have been more than 6800 events in these three days, and we see a spike at the end of the logs. However, even though we used the time filter for the day end on the 1st of December, we see no events after successful PowerShell execution. There have also been a lot more authentication events in the previous days than on the 1st of December.

8. To understand the events further, let's filter for our `user.name` with `service_admin` and `source.ip` with `10.0.11.11` to narrow our search.
![image](https://github.com/user-attachments/assets/ebaf3fd4-bb79-4909-abb2-a3592ec8ceb8)

Uh-oh! It looks like all these events have been coming from the same user and the same IP address. We definitely need to investigate further. This also does not explain the spike.

9. Let's filter for authentication events first by clicking the plus (+) button beside it.
10. Moreover, let's filter out the Source IP here to see if we can find the IP address that caused the spike. This can be done by clicking the minus (-) button beside it. After applying the filters, the expected result will be similar to the image below.
![image](https://github.com/user-attachments/assets/9c0a9dd7-8286-4cab-a60e-4d21b34cde3b)

![image](https://github.com/user-attachments/assets/99dc8291-3447-4a7b-85b9-34e201883a41)

Scrolling down, we see many events for failed logins. We also see that the IP address for the spike (ending in **.255.1**) differs from the one we saw for the events continuously coming in the previous days (10.0.11.11). The analysts have previously investigated this and found that a script with expired credentials was causing this issue. However, that script was updated with a fresh set of credentials. Anyhow, this might just be another script. Let's find out.

11. Let's remove the `source IP` filter so we can focus on authentication events close to the spike. After applying the new filter, we see that the failed logins stopped a little while after the successful login from the new IP.
![image](https://github.com/user-attachments/assets/ca4df012-4db6-4bb6-a1bb-80ffbddc21b2)

Our suspicions are rising. It seems that someone tried a brute-force attack on December 1st, as shown by the same filters applied above.
![image](https://github.com/user-attachments/assets/d865326b-886c-431b-8435-1ec570b38645)

The results also showed that they succeeded with the brute-force attempt because of the successful authentication attempt and quickly ran some PowerShell commands on the affected machines. Once the PowerShell commands were run, we didn't see any further login attempts. This looks like a **TP**, and there needs to be an escalation so that McSkidy can help us respond to this incident.

### Christmas in Danger?

The alarms have gone off, and McSkidy has been called to help take this incident further. McSkidy observed that nobody had actually looked at what the PowerShell command contained. Since the command was encoded, it needs to be decoded.
1. McSkidy changed the filters with `event.category: process` to take a deeper look at the PowerShell commands.
![image](https://github.com/user-attachments/assets/976953df-d3b9-4d92-9d80-72d6eebd78ae)

We can see the PowerShell command in the `process.command_line` field. 

`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -EncodedCommand SQBuAHMAdABhAGwAbAAtAFcAaQBuAGQAbwB3AHMAVQBwAGQAYQB0AGUAIAAtAEEAYwBjAGUAcAB0AEEAbABsACAALQBBAHUAdABvAFIAZQBiAG8AbwB0AA==`

McSkidy knows that Encoded PowerShell commands are generally Base64 Encoded and can be decoded using tools such as [CyberChef](https://gchq.github.io/CyberChef/). 

2. Since the command might contain some sensitive information and, therefore, must not be submitted on a public portal, McSkidy spins up her own instance of CyberChef hosted locally. McSkidy started by pasting the encoded part of the command in the Input pane in CyberChef.
3. Since it is a Base64 encoded command, McSkidy used two recipes, named "From Base64" and "Decode text" from the left pane. Note that McSkidy configured the **Decode text** to **UTF-16LE (1200)** since it is the encoding used by PowerShell for Base64.
![image](https://github.com/user-attachments/assets/62147932-395c-4588-9262-9753b7bc8a95)

The result provided a sigh of relief to McSkidy, who had feared that the Christmas had been ruined. Someone had come in to help McSkidy and the team secure their defences, but who?

### Villain or Hero?

McSkidy further analysed the secret hero and came to a startling revelation. The credentials for the script in the machines that ran the Windows updates were outdated. Someone brute-forced the systems and fixed the credentials after successfully logging in. This was evident from the fact that each executed PowerShell command was preceded by a successful login from the same Source IP, causing failed logins over the past few days. And what's even more startling? It was Glitch who accessed **ADM-01** and fixed the credentials after McSkidy confirmed who owned the IP address.
![image](https://github.com/user-attachments/assets/dec00e54-5f2a-4118-a442-d903e9722ee8)

### Answers
1. What is the name of the account causing all the failed login attempts? **service_admin**
2. How many failed logon attempts were observed? **6791**
![image](https://github.com/user-attachments/assets/c4b8810c-45c8-4d82-8fbb-46db39ae190a)

3. What is the IP address of Glitch? **10.0.255.1**
4. When did Glitch successfully logon to ADM-01? **Dec 1, 2024 08:54:39.000**
5. What is the decoded command executed by Glitch to fix the systems of Wareville? **Install-WindowsUpdate -AcceptAll -AutoReboot**
