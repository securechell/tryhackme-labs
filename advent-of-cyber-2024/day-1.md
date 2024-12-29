# Day 1: Maybe SOC-mas music, he thought, doesn't come from a store?

**Learning Objectives**
- Learn how to investigate malicious link files.
- Learn about OPSEC and OPSEC mistakes.
- Understand how to track and attribute digital identities in cyber investigations.

### Investigating the Website
The website we are investigating is a YouTube to MP3 converter currently being shared amongst the organizers of SOC-mas. You've decided to dig deeper after hearing some concerning reports about this website.

#### YouTube to MP3 Converter Websites
These popular sites have been around for a long time. However, historically, these websites have been observed to have risks, such as:

- **Malvertising**: Many sites contain malicious ads that can exploit vulnerabilities in a user's system, which could lead to infection.
- **Phishing scams**: Users can be tricked into providing personal or sensitive information via fake surveys or offers.
- **Bundled malware**: Some converters may come with malware, tricking users into unknowingly running it.

### Getting Some Tunes

*What nefarious thing does this website have in store for us??*

1. Let's find out by pasting any YouTube link in the search form and pressing the "Convert" button. Then select either `mp3 or mp4` option. This should download a file that we could use to investigate.

2. Once downloaded, navigate to the /root directory. Locate the file named `download.zip`, right-click on it, and select **Extract To**. In the dialog window, click the **Extract** button to complete the extraction.

3. You'll now see two extracted two files: `song.mp3` and `somg.mp3`. In Terminal run the `file` command on each one.

![image](https://github.com/user-attachments/assets/208d3fdb-77cb-41d4-8da2-d4702dd11aaa)

- `song.mp3`: Doesn't seem suspicious, according to the output. As expected, this is just an MP3 file.
- `somg.mp3`: From the filename alone, we can tell something is not right! Running the `file` command tells us that instead of an MP3, the file is an "MS Windows shortcut", also known as a `.lnk` file. This file type is used in Windows to link to another file, folder, or application. These shortcuts can also be used to run commands!

4. There are multiple ways to inspect `.lnk`  files to reveal the embedded commands and attributes. For this room we'll use **ExifTool**. Go to Terminal and type: `exiftool somg.mp3`

![image](https://github.com/user-attachments/assets/ddcb6694-b162-4342-8ad8-d36d6848586b)

Look through the output to locate the command used as a shortcut in the `somg.mp3` file. If you scroll down through the output, you should see a PowerShell command.

![image](https://github.com/user-attachments/assets/fc18c69b-ece2-4056-856d-a82ef51915bb)

What this PowerShell command does:
- The `-ep Bypass -nop` flags disable PowerShell's usual restrictions, allowing scripts to run without interference from security settings or user profiles.
- The `DownloadFile` method pulls a file (in this case, `IS.ps1`) from a remote server (`https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1`) and saves it in the `C:\\ProgramData\\` directory on the target machine.
- Once downloaded, the script is executed with PowerShell using the `iex` command, which triggers the downloaded `s.ps1` file.

If you visit the contents of the file to be downloaded using your browser (`https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1`), you will see just how lucky we are that we are not currently using Windows:

![image](https://github.com/user-attachments/assets/07c03976-a330-4407-a4d2-f72790738b83)

The script is designed to collect highly sensitive information from the victim's system, such as cryptocurrency wallets and saved browser credentials, and send it to an attacker's remote server.

This looks fairly typical of a PowerShell script for such a purpose, with one notable exception: a signature in the code that reads: **Created by the one and only M.M.**

### Searching the Source
There are many paths we could take to continue our investigation. We could investigate the website further, analyse its source code, or search for open directories that might reveal more information about the malicious actor's setup. We can search for the hash or signature on public malware databases like VirusTotal or Any.Run. Each of these methods could yield useful clues.

However, for this room, we'll try something a bit different. Since we already have the PowerShell code, searching for it online might give us useful leads. It's a long shot, but we'll explore it in this exercise.

There are many places where we can search for code. The most widely used is Github. So let's try searching there.

To search effectively, we can look for unique parts of the code that we could use to search with. The more distinctive, the better. For this scenario, we have the string we've uncovered before that reads: **"Created by the one and only M.M."**

1. Search for this on Github.com or by going directly to this link: https://github.com/search?q=%22Created+by+the+one+and+only+M.M.%22&type=issues.

![image](https://github.com/user-attachments/assets/98d88e7c-0149-4c4c-8210-f8aa9240f8a9)

2. You'll notice something interesting if you explore the pages in the search results. If you look through the search results, you can be able infer the malicious actor's identity based on information on the project's page and the GitHub Issues section.

![image](https://github.com/user-attachments/assets/b9eea057-ffd0-4726-a3f9-3e3819fd6795)

Aha! Looks like this user has made a critical mistake. This is a classic case of **OPSEC failure**.

### Introduction to OPSEC
**Operational Security** (OPSEC) is a term originally coined in the military to refer to the process of protecting sensitive information and operations from adversaries. The goal is to identify and eliminate potential vulnerabilities before the attacker can learn their identity.

When malicious actors fail to follow proper OPSEC practices, they might leave digital traces that can be pieced together to reveal their identity, such as:
- Reusing usernames, email addresses, or account handles across multiple platforms. One might assume that anyone trying to cover their tracks would remove such obvious and incriminating information, but sometimes, it's due to vanity or simply forgetfulness.
- Using identifiable metadata in code, documents, or images, which may reveal personal information like device names, GPS coordinates, or timestamps.
- Posting publicly on forums or GitHub (Like in this current scenario) with details that tie back to their real identity or reveal their location or habits.
- Failing to use a VPN or proxy while conducting malicious activities allows law enforcement to track their real IP address.

### Uncovering "MM"
If you've thoroughly investigated the GitHub search result, you should have uncovered several clues based on poor OPSEC practices by the malicious actor.

### Answers
1. Looks like the song.mp3 file is not what we expected! Run "exiftool song.mp3" in your terminal to find out the author of the song. Who is the author? **Tyler Ramsbey**.
2. The malicious PowerShell script sends stolen info to a C2 server. What is the URL of this C2 server? (Hint: Look for the "**$c2Url**" variable in the PowerShell script). **`http://papash3ll.thm/data`**.
3. Who is M.M? Maybe his Github profile page would provide clues? **Mayor Malware**.
4. What is the number of commits on the GitHub repo where the issue was raised? (Hint: Check out the commit history here: https://github.com/Bloatware-WarevilleTHM/CryptoWallet-Search/commits/main/). **1**.
