# Day 20: If you utter so much as one packet…

**Learning Objectives**
- Investigate network traffic using Wireshark
- Identify indicators of compromise (IOCs) in captured network traffic
- Understand how C2 servers operate and communicate with compromised systems

### Investigating the Depths
Before we dig deeper into Mayor Malware's intentions, we must learn a few essential things about C2 communication. **Command and Control (C2) Infrastructure** are a set of programs used to communicate with a victim machine. Whenever a machine is compromised, the C2 server drops its secret agent (**payload**) into the target machine. This secret agent is meant to obey the instructions of the C2 server. These instructions include executing malicious commands inside the target, exfiltrating essential files from the system, and much more. Interestingly, after getting into the system, the secret agent, in addition to obeying the instructions sent by the C2, has a way to keep the C2 updated on its current status. It sends a packet to the C2 every few seconds or even minutes to let it know it is active and ready to blast anything inside the target machine that the C2 aims to. These packets are known as **beacons**.

For this room, we will be using **Wireshark**, an open-source tool that captures and inspects network traffic saved as a **PCAP** file. It is beneficial for understanding the communications between a compromised machine and a C2 server.

>[!note]
>**Packet capture (PCAP)** is a networking practice involving the interception of data packets travelling over a network. Once the packets are captured, they can be stored by IT teams for further analysis. The inspection of these packets allows IT teams to identify issues and solve network problems affecting daily operations.

Here are some key capabilities you’ll see in this room:
- Wireshark can analyse traffic and display the information in an easy-to-navigate format regardless of the protocols used (e.g., HTTP, TCP, DNS).
- Wireshark can reconstruct back-and-forth conversations in a network.
- Wireshark allows easy filtering to narrow down essential details.
- Wireshark can also export and analyse objects that are transferred over the network.

### Diving Deeper
1. Now that we have a better idea of what C2 traffic looks like and how to use Wireshark, double-click on the file “_C2_Traffic_Analysis_” on the Desktop. This will automatically open the PCAP file using Wireshark. 
	- That's traffic! Yes, and this would take us to the truth about Mayor Malware.

2. We already suspect that this machine is compromised. So, let’s narrow down our list so that it will only show traffic coming from the IP address of Marta May Ware’s machine. To do this, click inside the **Display Filter Bar** on the top, type `ip.src == 10.10.229.217`, and press **Enter**.

![image](https://github.com/user-attachments/assets/0e11d967-f9f5-4538-aca1-97c3918c6519)

It’s still a lot, but at least we can now focus our analysis on outbound traffic.

3. If you scroll down a bit, you will find some interesting packets, specifically those highlighted in yellow:

![image](https://github.com/user-attachments/assets/d62e86e6-b649-47c9-b07c-eb09da65e7c7)

"Initial"? "Command"? "Exfiltrate"? That is sure to be something!

Let’s dive deeper.

### Message Received
1. If you click on the `POST /initial` packet (Frame 440), more details will be shown on the bottom panes. These panes will show more detailed information about the packet frame. The bottom-left pane shows relevant details such as frame number (440), the destination IP (10.10.123.224), and more.

![image](https://github.com/user-attachments/assets/29a0318d-29ce-4dbf-92a6-c848bf80e102)

You can expand each detail if you want, but the critical area to focus on is the lower-right view, the “Packet Bytes” pane.

This pane shows the bytes used in the communication in hexadecimal and ASCII character formats. The latter format shows readable text, which can be helpful in investigations.

The screenshot above shows something interesting: “I am in Mayor!”. This piece of text is likely relevant to us. This is a message sent by a payload that Mayor Malware has sent into Martha May Ware's computer, that is now communicating back to Mayor Malware (the C2 server) basically saying: I'm here.

2. If we right-click on the `POST /initial` packet (Frame 440) and select Follow -> HTTP Stream, a new pop-up window will appear containing the back-and-forth HTTP communication relevant to the specific session.

![image](https://github.com/user-attachments/assets/5421513c-3799-45ee-a97b-4a6156e7fdf7)

![image](https://github.com/user-attachments/assets/cf7e5775-317e-4d6e-bd53-bcdef3bbb3f6)

This feature is useful when you need to view all requests and responses between the client and the server, as it helps you understand the complete context of the communication.

The text highlighted in red is the message sent from the source (Martha) to the destination (Mayor), and blue is from the destination to the source. So, based on the screenshot above, we can see that after the message “I am in Mayor!” was sent, a response that reads “Perfect!" was sent back (the C2 server is acknowledging that the payload is now inside the victim's computer).

_Perfect_, indeed, Mayor. We got you now!

3. But let’s not stop here. Other interesting HTTP packets were sent to the same destination IP. If you follow the HTTP Stream for the `GET /command` packet (Frame 457), you’ll see a request to the same IP destination (just click arrow in Stream in bottom-right corner to go to Stream 2). 

![image](https://github.com/user-attachments/assets/49bd89e3-959f-4a16-9111-5fefc2aed80e)

Interestingly, the reply ("whoami") that came back was a command commonly used in Windows and Linux systems to display the current user’s information. This communication suggests that the destination is attempting to gather information about the compromised system, a typical step during an early reconnaissance stage.

Usually, the reply from a C2 server contains the command, instructing the malicious program what to do next. However, the type of instruction depends on the malicious actor’s configuration, intention, and capabilities. These instructions often fall into several categories:

  1. **Getting system information:** The attacker may want to know more about the compromised machine to tailor their next moves. *This is what we are seeing above*.
  2. **Executing commands:** If the attacker needs to perform specific actions, they can also send commands directly. However, this is less stealthy and easily attracts attention.
  3. **Downloading and executing payloads:** The attacker can also send additional payloads to the machine containing additional functionality or tools.
  4. **Exfiltrating data:** This is one of the most common objectives. The program may be instructed to steal valuable data such as sensitive files, credentials, or personal information.

"Exfiltrate" sounds familiar, right?

### Exfiltrating the Package
1. If we follow the HTTP Stream for the `POST /exfiltrate` packet (Frame 476) sent to the same destination IP, we will see a file exfiltrated to the C2 server. We can also find some clues inside this file.

![image](https://github.com/user-attachments/assets/0067b929-b8bc-4756-a1e9-79e35c698655)

It seems a file called "credentials.txt" was sent by the payload, along with the message "AES ECB is your chance to decrypt the encrypted beacon with the key: 1234567890abcdef1234567890abcdef
--f5964f77-daf1-4853-aacb-df4754eaacaf--". Mayor Malware responded with "Data received".

2. Check the rest of the PCAP, you’ll find that more interesting packets were captured. Let’s break these down and dive deeper into what we’ve uncovered.

3. Click to see the next Stream. We can see that a message with an encrypted beacon (8724670c271adffd59447552a0ef3249) was sent and acknowledged.

![image](https://github.com/user-attachments/assets/d45e39a0-b510-483a-a5db-30f9e10b063c)

4. Continue through the Streams. We can see that the same encrypted message is being sent (and acknowledged).

### What’s in the Beacon
A typical C2 beacon returns regular status updates from the compromised machine to its C2 server. The beacons may be sent after regular or irregular intervals to the C2 as a heartbeat. Here’s how this exchange might look:

- **Secret agent (payload):** “I am still alive. Awaiting any instructions. Over.”
- **C2 server:** “Glad to hear that! Stand by for any further instructions. Over.”

In this scenario, Mayor Malware’s agent (payload) inside Marta May Ware’s computer has sent a message that is sent inside all the beacons. Since the content is highly confidential, the secret agent encrypts it inside all the beacons, leaving a clue for the Mayor’s C2 to decrypt it. In the current scenario, we can identify the beacons by the multiple requests sent to the C2 from the target machine after regular intervals of time.

The exfiltrated file's content hints at how these encrypted beacons can be decrypted. Using the encryption algorithm with the provided key, we now have a potential way to unlock the beacon’s message and uncover what Mayor Malware's agent is communicating to the C2 server.

But what exactly are we about to reveal?

Since the beacon is now encrypted and you have the key to decrypt it, the [CyberChef](https://gchq.github.io/CyberChef/) tool would be our source of truth for solving this mystery. Because of its wide features, CyberChef is considered a "Swiss Army Knife". We can use this tool for encoding, decoding, encrypting, decrypting, hashing, and much more. However, considering this task's scope, we would only cover the decryption process using this tool.

1. Open the CyberChef tool in your browser.
2. From the tool's dashboard, you will utilise panes for decrypting your beacon.
3. **Operations:** Search for AES Decrypt and drag it to the **Recipe** area, which is in the second pane.
4. **Recipe:** Select the mode of encryption as ECB, and enter the decryption key (1234567890abcdef1234567890abcdef). Keep the other options as they are.
5. **Input:** Enter the encrypted beacon (8724670c271adffd59447552a0ef3249) by copy-pasting the encrypted string here.
6. **Output:** Once you have completed the above steps, click "Bake". Your encrypted string will be decrypted using the AES ECB decryption with the key you provided, and the output (flag) will be displayed.

![image](https://github.com/user-attachments/assets/11dfee05-6e12-4b93-aa8e-3824cd186203)


### Answers
1. What was the first message the payload sent to Mayor Malware’s C2? (Hint: Look at the `POST /initial` packet.) **I am in Mayor!**
2. What was the IP address of the C2 server? **10.10.123.224**

![image](https://github.com/user-attachments/assets/1b2772f5-15be-49e3-8d08-99956f30f04e)

3. What was the command sent by the C2 server to the target machine? (Hint: Look at the `GET /command` packet). **whoami**
4. What was the filename of the critical file exfiltrated by the C2 server? (Hint: Look at the POST / exfiltrate packet). **credentials.txt**
5. What secret message was sent back to the C2 in an encrypted format through beacons? (Hint: Check the beacon traffic and use the clue from the exfiltrated file). **THM_Secret_101**
