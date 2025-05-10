# Day 11: If you'd like to WPA, press the star key!

**Learning Objectives**
- Understand what Wi-Fi is
- Explore its importance for an organisation
- Learn the different Wi-Fi attacks
- Learn about the WPA/WPA2 cracking attack

### What is Wi-Fi
Wi-Fi is the technology that connects our devices to the global network, the Internet.
To connect to Wi-Fi, we turn it on from our devices, and it lists all the available Wi-Fi networks around us. Once you successfully connect to a network via Wi-Fi, you will be assigned an IP address inside that network, which will uniquely identify you and help you communicate with other devices. It is just like becoming a member of a family assigned with a name that the whole family trusts.

With all this discussion on Wi-Fi, it seems like a door to our internet access, and every Wi-Fi connection forms a family of devices. Would you allow somebody you don't really know to become part of your family? Not that easy! Probably because of the privileges a family member has, nobody from outside should ever get those.

### Wi-Fi's Pivotal Role in Organisations
Most organisations rely on the Internet for their business functioning. They use Wi-Fi for their networks to connect their employees to the Internet. As the employees connect to the organisation's network, they form a family of interconnected devices that can communicate with each other. Organisations tend to recruit trustworthy and professional employees to avoid any misuse of their privileges inside the network.

However, a malicious actor from outside the organisation could still see the broadcasted Wi-Fi **SSID** of the organisation's network when they turn their Wi-Fi on. This may not seem to be a problem as the attacker does not know the password, but the attacker actually has some other plans as well!

### The Practical
#### WPA/WPA2 Cracking
1. Access the **VM** from the AttackBox via SSH with the following command `ssh glitch@10.10.165.150` and entering the credentials:
	- **Username**: glitch
	- **Password**: Password321
	- **IP**: 10.10.165.150

2. On the current SSH session, run the command `iw dev`. This will show any wireless devices and their configuration that we have available for us to use.

![image](https://github.com/user-attachments/assets/1be6b199-1b55-452f-bfcd-b3b444989f56)

- The device/interface `wlan2` is available to us, and there are two important details to take away from this output that will be useful to us:
	- The `addr` is the **MAC/BSSID** of our device. BSSID stands for Basic Service Set Identifier, and it's a unique identifier for a wireless device or access point's physical address.
	- The `type` is shown as **managed**. This is the standard mode used by most Wi-Fi devices (like laptops, phones, etc.) to connect to Wi-Fi networks. In managed mode, the device acts as a client, connecting to an access point to join a network. There is another mode called **monitor**, which we will discuss shortly.

3. Now, we would like to scan for nearby Wi-Fi networks using our `wlan2` device. Type `sudo iw dev wlan2 scan`.
	- `dev wlan2` specifies the wireless device you want to work with
	- `scan` tells `iw` to scan the area for available Wi-Fi networks.

![image](https://github.com/user-attachments/assets/653a014d-e0dd-4937-b7c7-79100b43712d)

Now will be a good time to discuss another type that we can use on some wireless devices: **monitor** mode. This is a special mode primarily used for network analysis and security auditing. In this mode, the Wi-Fi interface listens to *all* wireless traffic on a specific channel, regardless of whether it is directed to the device or not. It passively captures all network traffic within range for analysis without joining a network. 

4. We want to check if our `wlan2` interface can use monitor mode. Run the command `sudo ip link set dev wlan2 down` to turn our device off.
5. Then switch modes with `sudo iw dev wlan2 set type monitor` to change **wlan2** to **monitor** mode. 
6. We will then turn our device back on with `sudo ip link set dev wlan2 up`.
7. We can confirm that our interface is in monitor mode with the command `sudo iw dev wlan2 info`.

![image](https://github.com/user-attachments/assets/4ebdf0f2-dbbe-4259-82f2-4e9e98d38322)

8. Now, let us create one more SSH session. We want to have **2 separate terminals** in order to see clearly how the attack works. You can align the SSH terminals however you like, but here is an example of how it should look:

![image](https://github.com/user-attachments/assets/03ada0c9-8bac-46f6-a0f5-f7b591d1765a)

9. On the **first** terminal, we start by capturing Wi-Fi traffic in the area, specifically targeting the WPA handshake packets. Run the command `sudo airodump-ng wlan2`. This command provides a list of nearby Wi-Fi networks (SSIDs) and shows important details like signal strength, channel, and encryption type. This information is already known to us from our previous commands.

>[!note]
>By default, `airodump-ng` will automatically switch the selected wireless interface into monitor mode if the interface supports it.

![image](https://github.com/user-attachments/assets/81942ea1-0574-4ba0-8126-3c04bc37c1bc)

The output reveals the information we already knew before, such as the BSSID, SSID, and the channel. However, in this particular output, we are also given the channel where our target SSID is listening (channel 6). Now, we will focus on the **MalwareM_AP** access point and capture the WPA handshake; this is crucial for the PSK (password) cracking process.

10. In the **first** terminal, let us cancel **airodump-ng** using **CTRL+C** and then execute the command `sudo airodump-ng -c 6 --bssid 02:00:00:00:00:00 -w output-file wlan2`. 
	- This command targets the specific network channel and MAC address (BSSID) of the access point for which you want to capture the traffic and saves the information to a few files that start with the name **output-file**. These files will be used to crack the PSK. 
	- The ultimate goal of this command is to capture the 4-way handshake. It will check for any clients connected to the access point. If a client is already connected, then we can perform a deauthentication attack; otherwise, for any new client that connects, we will capture the 4-way handshake. 
	- In this particular scenario, a client is already connected. The output will look the same at first until we receive the information about the connected client, which will be displayed at the bottom of our output. It is important to leave this **command running** until we are done with the attack.
	- It should take between **1 to 5 minutes** before receiving the client information.

![image](https://github.com/user-attachments/assets/534f573a-74dd-4d69-b665-620d708308d0)

Note that the `STATION` section shows the device's BSSID (MAC) of `02:00:00:00:01:00` that is connected to the access point. This is the connection that we will be attacking. 

11. On the **second** terminal, we will launch the deauthentication attack. Because the client is already connected, we want to force them to reconnect to the access point, forcing it to send the handshake packets. We can break this down into 3 simple steps:
	1. **Deauthentication packets:** The tool aireplay-ng sends deauthentication packets to either a specific client (targeted attack) or to all clients connected to an access point (broadcast attack). These packets are essentially "disconnect" commands that force the client to drop its current Wi-Fi connection.
	2. **Forcing a reconnection:** When the client is disconnected, it automatically tries to reconnect to the Wi-Fi network. During this reconnection, the client and access point perform the 4-way handshake as part of the reauthentication process.
	3. **Capturing the handshake:** This is where airodump-ng comes into play because it will capture this handshake as it happens, providing the data needed to attempt the WPA/WPA2 cracking.

We can do this with `sudo aireplay-ng -0 1 -a 02:00:00:00:00:00 -c 02:00:00:00:01:00 wlan2`. 
- `-0` flag indicates that we are using the deauthentication attack
- The `1` value is the number of deauthentications to send.
- `-a` indicates the BSSID of the access point
- `-c` indicates the BSSID of the client to unauthenticated

![image](https://github.com/user-attachments/assets/71a20510-9ac4-4fff-9cc7-7438be549470)

12. Now, if we look back on our **first** terminal, we will see the WPA handshake shown on the top-right of our output as `WPA handshake: 02:00:00:00:00:00`. All of this information is being saved into our output files.

![image](https://github.com/user-attachments/assets/758d4047-613f-4a10-809e-0c84fb22c23a)

13. In the **second** terminal, we can use the captured WPA handshake to attempt to crack the WPA/WP2 passphrase. We will be performing a dictionary attack in order to match the passphrase against each entry in a specified wordlist file. A shortened version of the infamous `rockyou.txt` wordlist has already been provided for us to use. This is located in the `/home/glitch/` directory. If the passphrase is weak and appears in the wordlist, it will eventually be cracked. The command `sudo aircrack-ng -a 2 -b 02:00:00:00:00:00 -w /home/glitch/rockyou.txt output*cap` will do this for us, where:
	- `-a 2` flag indicates the WPA/WPA2 attack mode
	- `-b` indicates the BSSID of the access point
	- the `-w` flag indicates the dictionary list to use for the attack. 

14. Now we select the output files that we will be using, which contain the 4-way handshake that we will be cracking.

![image](https://github.com/user-attachments/assets/a11db5d5-1d41-432d-b4b6-944658da77da)

>[!note]
>If you get a `Packets contained no EAPOL data; unable to process this AP` error, this means that you ran aircrack-ng prior to the handshake being captured or that the handshake was not captured at all. If that's the case, then re-do all of the steps in order to capture the `WPA handshake`.

15. With the PSK, we can now join the **MalwareM_AP** access point. In a typical engagement, we would do this to inspect the new network, or in some cases, joining the access point is enough to show impact. Press **CTRL+C** on the terminal that has `airodump-ng` running (the **first** terminal) in order to stop the **airodump-ng** process. We do this because we will not be able to join the Wi-Fi network while airodump-ng is running due to the fact that we are actively using the interface in monitor mode. 

16. Execute the following commands:

![image](https://github.com/user-attachments/assets/c97501fb-4c42-4d5f-8d7e-361e3fb3536f)

>[!note]
>If you get a `rfkill: Cannot get wiphy information` error, you can ignore it. You will also notice that `wpa_supplicant` has automatically switched our **wlan2** interface to **managed mode**.

17. Give it about **10 seconds** and checking both wireless interfaces once again with `iw dev`. The terminal will show that we have joined the **MalwareM_AP** SSID.

![image](https://github.com/user-attachments/assets/39137206-5522-4947-bad6-13ecec08a178)

### Answers
1. What is the BSSID of our wireless interface?**02:00:00:00:02:00**.

![image](https://github.com/user-attachments/assets/455f14d2-5947-480f-8a5e-0da0d49d58e2)

2. What is the SSID and BSSID of the access point? Format: SSID, BSSID. **MalwareM_AP, 02:00:00:00:00:00**.

![image](https://github.com/user-attachments/assets/709404eb-92a6-481f-a136-305b56d1027f)

3. What is the BSSID of the wireless interface that is already connected to the access point? **02:00:00:00:01:00**.
4. What is the PSK after performing the WPA cracking attack? **fluffy/champ24**.

![image](https://github.com/user-attachments/assets/d6c0baf7-56f7-49cf-9ff7-40c61635c075)
