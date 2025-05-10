# Day 21: HELP ME...I'm REVERSE ENGINEERING!

**Learning Objectives**
- Understanding the structure of a binary file 
- The difference between Disassembly vs Decompiling
- Familiarity with multi-stage binaries
- Practically reversing a multi-stage binary

### Introduction to Reverse Engineering
**Reverse Engineering** (RE) is the process of breaking something down to understand its function. In cyber security, reverse engineering is used to analyse how applications (binaries) function. This can be used to determine whether or not the application is malicious or if there are any security bugs present.

For example, cyber security analysts reverse engineer malicious applications distributed by attackers to understand if there are any attributable indicators to associate the binary with an attacker and any potential ways to defend against the malicious binary.

### Binaries
﻿In computing, binaries are files compiled from source code. For example, you run a binary when launching an executable file on your computer. At one point in time, this application would've been programmed in a programming language such as C#. It is then compiled, and the compiler translates the code into machine instructions.

Binaries have a specific structure depending on the operating system they are designed to run. For example, Windows binaries follow the Portable Executable (PE) structure, whereas on Linux, binaries follow the Executable and Linkable Format (ELF). This is why, for example, you cannot run a **.exe** file on MacOS. With that said, all binaries will contain at least:
- **A code section:** This section contains the instructions that the CPU will execute
- **A data section:** This section contains information such as variables, resources (images, other data), etc
- **Import/Export tables:** These tables reference additional libraries used (imported) or exported by the binary. Binaries often rely on libraries to perform functions. For example, interacting with the Windows API to manipulate files

The binaries in today's task follow the **PE structure**.

### Disassembly Vs. Decompiling
When reverse engineering binaries, you will employ two primary techniques: **disassembly** and **decompiling**.

- **Disassembling** a binary shows the low-level machine instructions the binary will perform (you may know this as assembly). Because the output is translated machine instructions, you can see a detailed view of how the binary will interact with the system at what stage. 

- **Decompiling**, however, converts the binary into its high-level code, such as C++, C#, etc., making it easier to read. However, this translation can often lose information such as variable names. This method of reverse engineering a binary is useful if you want to get a high-level understanding of the application's flow.

There are specific circumstances where you would choose one method over the other. For example, decompiling is sometimes a "best guess" based on the tooling you've used and does not provide the actual full source code.

### Multi-Stage Binaries

Recent trends in cyber security have seen the rise of attackers using what's known as "Multi-stage binaries" in their campaigns - especially malware. These attacks involve using multiple binaries responsible for different actions rather than one performing the entire attack. Usually, an attack involving various binaries will look like the following:

1. **Stage 1 - Dropper:** This binary is usually a lightweight, basic binary responsible for actions such as enumerating the operating system to see if the payload will work. Once certain conditions are verified, the binary will download the second - much more malicious - binary from the attacker's infrastructure.
2. **Stage 2 - Payload:** This binary is the "meat and bones" of the attack. For example, in the event of ransomware, this payload will encrypt and exfiltrate the data.

Sophisticated attackers may further split actions of the attack chain (e.g., lateral movement) into additional binaries. Using multiple stages helps evade detection and makes the analysis process more difficult.

For example, a small, more "harmless" initial binary is likelier to evade detection via email filtering than a fully-fledged binary that performs malicious actions such as encryption. Additionally, splitting these functions into multiple stages gives the attacker much more control (i.e. only downloading specific stages once conditions such as time have been met).

### Jingle .NET all the way
For today's task, you will be reverse engineering two .NET binaries using the decompiler ILSpy. You can follow the walkthrough below in reverse engineering using an example application named `demo.exe`. Then, you will reverse an application on your own at the end of this task.

Before analysing our target, we need to learn and find a way to identify the original binary file, modify it, or use it as evidence. Also, it is good practice to have a big picture of the file we are dealing with so that we can choose the proper tools we will need.

1. Let's start by navigating to the file location in the **demo** folder on the machine's Desktop by right-clicking on the file named **demo** and clicking on Properties.
2. We can observe that the file's extension is .exe, indicating that it is a Windows executable.

![image](https://github.com/user-attachments/assets/4a6cabcb-d165-4989-80b2-7bb0568377d1)

3. Since it's a Windows file, we'll use **PEStudio**, a software designed to investigate potentially malicious files and extract information from them without execution. This will help us focus on the static analysis side of the investigation. Let's open PEstudio from the taskbar and then click on File -> Open File and select the file `demo.exe` located in `C:\Users\Administrator\Desktop\demo\demo.exe`.

4. PEStudio will display information about the file, so let's start enumerating some of the most important aspects we can get from it. Using the left panel, we can navigate through different sections that will share different types of information about the file. In the general information output displayed when opening the file, we can see the hash of the file in the form of **SHA-256**, The architecture type, in this case, **x64**, the file type, and the signature of the language used to compile the executable, in this case, .**NET framework** that uses the C# language.

![image](https://github.com/user-attachments/assets/de7f392d-09c2-44d1-bee9-46f769f9fbc0)

Let's focus on some critical data we can obtain. First, if we want to identify the file and provide evidence of its alteration, we need to take note of the file’s SHA-256 hash, as we mentioned above, as well as the hash of each section on the PE file. PE stands for Portable Executable, and it's the format in which Windows executables are arranged.

The sections represent a memory space with different content of the Windows executable within the PE format. We can calculate the hash of these sections in order to identify the executable properly. We'll focus this time on two hashes: the one from the .text section, which is the section on the file containing the executable code; this section is often marked as Read and executable, so it should not have any alterations when the file is being copied. We can observe them in the screenshot below:

![image](https://github.com/user-attachments/assets/d81a186b-7891-4429-b071-7a52a95268a5)

Another essential section we can use to obtain information about the binary is the "indicators" section. This section tells us about the potential indicators like URLs or other suspicious attributes of a binary.

![image](https://github.com/user-attachments/assets/6970c220-7006-4450-98df-220447a95589)

The screenshot above shows several strings on the file, like file names, URLs, and function names. This can be very important depending on the file's execution flow. Additionally, looking for artefacts such as IP addresses, URLs, and crypto wallets can be a "quick win" for gathering some identifying intelligence.

Now that we have information about the file we are investigating, let's try to understand what the executable is doing. We need to understand its flow. If we try to read the file by opening it, we cannot do it since it's in binary format. In the previous section, we learned that the file is compiled using the .NET framework used by the C# language. We can decompile the binary code into C# using a decompilation tool like **ILSpy**.

This tool will decompile the code, providing us with readable information we can use to determine the flow of execution. 

5. Let's start by opening ILSpy from the taskbar, click on File -> Open, then navigate to `C:\Users\Administrator\Desktop\demo` and select the file `demo.exe`. The tool ILSpy may take up to 30 seconds to appear on the screen.

![image](https://github.com/user-attachments/assets/5f22554b-625c-45f4-a7be-7020d782a2f1)

6. As we can observe from above, the left panel contains the libraries used by the framework, and the actual decompiled code is in the section with the file name **demo**. Let's click on it to expand and see what it contains.

![image](https://github.com/user-attachments/assets/b7d244bc-4e00-483a-8d0a-1051d4164e30)

As the screenshot above shows, ILSpy can provide much information, like metadata and references. However, the actual show is displayed on the brackets symbols {}, in this case, under DemoBinary -> Program -> Main, which is actually the Main function of the executable. Now that we have access to the code running on the binary, it can be analysed.

7. Analysis indicates the binary will download a PNG file to the user's Desktop from the URL: `http://10.10.10.10/img/tryhackme_connect.png`. Let's execute the file and see if this is true: Double-click on the `demo.exe` file.

8. Once the binary starts, wait for the text "`Hello THM DEMO Binary`" to appear, then press **Enter** to download the file.

![image](https://github.com/user-attachments/assets/0b01f52d-65a2-40aa-ad70-d4404690ed99)

![image](https://github.com/user-attachments/assets/8b4e0482-b328-4e13-8416-b72d75001d71)

9. After executing the file, we can observe that `thm-demo` was downloaded to the Desktop, and the messages print to the screen as expected. Also, the downloaded file is executed and opened using the default app for PNG Paint. Excellent, we successfully Reverse-Engineered the flow of the code.

![image](https://github.com/user-attachments/assets/89505671-263b-4040-8b82-1ec1989260ef)

### Practical

Now that we have some practice, join McSkidy and help investigate the alerts coming from the file _WarevilleApp.exe_. Put on your reverse engineering hat and help decipher the mystery behind this suspicious binary.

To answer the question, you will need to reverse the application `WarevilleApp.exe`, located at `C:\Users\Administrator\Desktop\`.

1. Find out the Properties of the WarevilleApp file. We can see it's a Windows executable (.exe).

![image](https://github.com/user-attachments/assets/455f7e61-c133-49f4-86b8-f9abb732d012)

2. We'll use PEStudio to investigate. Open PEstudio and select the file `WarevilleApp.exe`.

3. We can see some general info about the file.

![image](https://github.com/user-attachments/assets/bdcf3885-9043-4058-ac69-0cd3e92f6147)

4. Open ILSpy and select `WarevilleApp.exe`. references. Go to FancyApp -> Program -> Main.

![image](https://github.com/user-attachments/assets/0348ecc3-234f-479c-b4ab-61462b5aa9da)

5. The code can be analysed.

### Answers
1. What is the function name that downloads and executes files in the `WarevilleApp.exe`? (Hint: Use the ILSpy tool, expand the Form1 section and inspect each function.) **DownloadAndExecuteFile**
	- Go to FancyApp -> Form1
	- Scroll down looking at each function.
	- The one that says "`DownloadAndExecuteFile`:

![image](https://github.com/user-attachments/assets/577c0180-6a7e-42c2-b1c0-d707a97fc2b5)

2. Once you execute the `WarevilleApp.exe`, it downloads another binary to the Downloads folder. What is the name of the binary? **explorer.exe**
	- Execute (double-click) `WarevilleApp.exe`
	- Go to Downloads and see what new file is there.

3. What domain name is the one from where the file is downloaded after running WarevilleApp.exe? **mayorc2.thm**
	- In ILSpy go to FancyApp -> Form1
	- Look under `DownloadAndExecuteFile()`. The domain name is there.

![image](https://github.com/user-attachments/assets/e22780f3-e452-4117-8dd6-d1032d3a307a)

4. The stage 2 binary is executed automatically and creates a zip file comprising the victim's computer data; what is the name of the zip file? (Hint: Visit the Pictures folder to check the zip file.) **CollectedFiles.zip**
	- Go to Pictures, `CollectedFiles` is there. For the correct answer you need to add the file extension!

5. What is the name of the C2 server where the stage 2 binary tries to upload files? (Hint: Use the ILSpy tool and check the DownloadAndExecuteFile function.) **anonymousc2.thm**
	- In ILSpy open `explorer.exe`
	- Go to FileCollector -> Program -> UploadFileToServer
	- The domain name shown is the C2 server

![image](https://github.com/user-attachments/assets/7a181a5f-20bc-4ff0-b5e5-3374ea5ca77f)
