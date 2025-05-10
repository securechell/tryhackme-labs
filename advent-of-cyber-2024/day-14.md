# Day 14: Even if we're horribly mismanaged, there'll be no sad faces on SOC-mas!

**Learning Objectives**
In today's task you will learn about:
- Self-signed certificates
- Man-in-the-middle attacks
- Using Burp Suite proxy to intercept traffic


### Certified to Sleigh

A **certificate** is composed of:

- **Public key**: At its core, a certificate contains a public key, part of a pair of cryptographic keys: a public key and a private key. The public key is made available to anyone and is used to encrypt data.
- **Private key**: The private key remains secret and is used by the website or server to decrypt the data.
- **Metadata**: Along with the key, it includes metadata that provides additional information about the certificate holder (the website) and the certificate. You usually find information about the Certificate Authority (CA), subject (information about the website, e.g.` www.meow.thm`), a uniquely identifiable number, validity period, signature, and hashing algorithm.

### Sign Here, Trust Me

#### So what is a Certificate Authority (CA)?
A CA is a trusted entity that issues certificates; for example, GlobalSign, Let’s Encrypt, and DigiCert are very common ones. The browser trusts these entities and performs a series of checks to ensure it is a trusted CA. Here is a breakdown of what happens with a certificate:

- **Handshake**: Your browser requests a secure connection, and the website responds by sending a certificate, but in this case, it only requires the public key and metadata.
- **Verification:** Your browser checks the certificate for its validity by checking if it was issued by a trusted CA. If the certificate hasn’t expired or been tampered with, and the CA is trusted, then the browser gives the green light.
- **Key exchange**: The browser uses the public key to encrypt a session key, which encrypts all communications between the browser and the website.
- **Decryption**: The website (server) uses its private key to decrypt the session key, which is symmetric. Now that both the browser and the website share a secret key (session key), we have established a secure and encrypted communication!

*Ever wonder what makes HTTPS be **S** (secure)? Thanks to certificates, we can now have authentication, encryption, and data integrity.*

#### Self-Signed Certificates vs. Trusted CA Certificates
The process of acquiring a certificate with a CA is long, you create the certificate, and send it to a CA to sign it for you. If you don’t have tools and automation in place, this process can take weeks. Self-signed certificates are signed by an entity usually the same one that authenticates. For example, Wareville owns the GiftScheduler site, and if they create a certificate and sign it with Wareville as a CA, that becomes a self-signed certificate.

- **Browsers** generally do not trust self-signed certificates because there is no third-party verification. The browser has no way of knowing if the certificate is authentic or if it’s being used for malicious purposes (like a **man-in-the-middle attack**).
- **Trusted CA certificates**, on the other hand, are verified by a CA, which acts as a trusted third party to confirm the website’s identity.

CA-issued certificates sometimes take a long time; if you want to test a development environment, it can make sense to use self-signed certificates. Ideally, this is an internal, air-gapped environment with no connection to the public Internet. Otherwise, it defeats the purpose of a certificate: the entire system of secure communication relies on the fact that both parties (the browser and the server) can trust the data being exchanged and that no one in the middle can intercept or modify it without detection.

### How Mayor Malware Disrupts G-Day
There are less than two weeks until G-Day, and Mayor Malware has been planning its disruption ever since Glitch raised the self-signed certificate vulnerability to McSkidy during a security briefing the other day.

His plan is near perfect. He will hack into the Gift Scheduler and mess with the delivery schedule. No one will receive the gift destined for them: G-Day will be ruined!

#### Preparation
First things first: the Glitch spoke about a self-signed certificate, but Mayor Malware can’t believe that the townspeople—usually so security-savvy it’s maddening to him—would easily disregard such a critical vulnerability. Is it a trap set up by the Glitch and McSkidy to catch him red-handed? He definitely needs to check for himself.

Before that, though, he wants to make sure that his tracks are well covered. To prevent any DNS logs from alerting his enemies, he will resolve the Gift Scheduler’s FQDN locally on his machine.

To achieve this, let’s add the following line to the `/etc/hosts` file on the AttackBox: `10.10.16.22 gift-scheduler.thm`

1. In Terminal type: `echo "10.10.16.22 gift-scheduler.thm" >> /etc/hosts`.
2. Execute `cat /etc/hosts` to verify that the line above was added to the file:
![image](https://github.com/user-attachments/assets/86dc137b-9e84-48ce-8eb4-43d32df4df94)

Now, Mayor Malware can navigate to the Gift Scheduler website without leaving a trace on Wareville’s DNS logs.

3. Open the Firefox browser and navigate to `https://gift-scheduler.thm`. We’ll be presented with a warning page. Click the Advanced button to expand the warning’s details.
![image](https://github.com/user-attachments/assets/f1376269-6692-44b0-8bd0-0b8676089155)

4. Click View Certificate - a new tab opens with the certificate details.
![image](https://github.com/user-attachments/assets/9fa11992-33a8-4810-bcb3-39378f752375)

Mayor Malware can’t believe his luck! This is evidence that the Glitch was speaking the truth: the Gift Scheduler web server uses a self-signed certificate.

This means that the townspeople and all the elves will be used to clicking on the "Accept the Risk and Continue" button to access the website, to the point it’s become a habit.

Mayor Malware does just that and inserts his credentials into the login form:
**Username**: mayor_malware, **Password**: G4rbag3Day

With his credentials, he can’t do anything but send a gift request—as if he were to ever do such a sickeningly sweet gesture. To carry out his evil plan, he will need to sniff some admin credentials. Maybe some of the elves’ passwords. Or even—if he gets lucky—Marta May Ware’s account!

To sniff the elves’ traffic, the next step will be to start a proxy on his machine and route all of Wareville’s traffic to it. This way, the **Mayor** will be **In The Middle** between the townspeople and the Gift Scheduler. This position will allow him to sniff all requests forwarded to the sickening website.

5. Start the Burp Suite proxy by typing `burp` in the terminal. A new window will open. Accept the default configuration by clicking on Next, then Start Burp.
6. Once Burp Suite loads, select Proxy, then turn off Intercept to prevent users from noticing any delays in the website responses.
7. Open the Proxy Settings to set a new listener on our AttackBox IP address.
	- Click Add; Burp Suite will prompt us for the new listener’s configuration.
	- Set the listening port to 8080 and toggle the Specific address option. The box next to it will automatically specify the IP address of our AttackBox, (`10.10.123.222`). Click OK to apply the configuration.
![image](https://github.com/user-attachments/assets/6a8faf1a-15ff-4255-9ac4-e49bfcafe398)

	- The previous settings window will get displayed and we can see that the new listener has been added under the proxy listeners list:
![image](https://github.com/user-attachments/assets/b920c10b-b1ba-4ed8-bd0a-1e8ed4a90815)

Mayor Malware rubs his hands together gleefully: as we can see in the screenshot above, Burp Suite already comes with a self-signed certificate. The users will be prompted to accept it and continue, and Mayor Malware knows they will do it out of habit, without even thinking of verifying the certificate origin first. The G-Day disruption operation will go off without a hitch!

#### Sniff From The Middle
Now that our machine is ready to listen, we must reroute all Wareville traffic to our machine.

Mayor Malware has a wonderful idea to achieve this: he will set his own machine as a gateway for all other Wareville’s machines!

1. Add another line to the AttackBox’s `/etc/hosts` file by typing the following command in the Terminal: `echo "10.10.123.222 wareville-gw" >> /etc/hosts`. This will divert all of Wareville’s traffic, usually routed through the legitimate Wareville Gateway, to Mayor Malware’s machine, effectively putting him *“In The Middle”* of the requests.
2. As a last step, we must start a custom script to simulate the users’ requests to the Gift Scheduler. On the AttackBox, the script can be found in `/root/Rooms/AoC2024/Day14`. **Note:** Keep the script running so that new user requests will constantly be captured in Burp Suite.
![image](https://github.com/user-attachments/assets/c5e5d67d-40fd-4930-8e7a-4bf1af5cdcd6)

#### Pwn the Scheduler
At last, everything is in place. Mayor Malware’s evil plan can finally commence!

1. Return to the open Burp Suite window and click on the HTTP History tab.
![image](https://github.com/user-attachments/assets/366730f6-3d26-4ada-84ae-c07301188d4d)

There is a triumphant gleam in Mayor Malware’s eyes while he stares intently at the web requests pouring on his screen. He can finally see them: the POST requests containing clear-text credentials for the Gift Scheduler website! Now, he only needs to wait and find the password to a privileged account.

### Answers
1. What is the name of the CA that has signed the Gift Scheduler certificate? **THM**.
![image](https://github.com/user-attachments/assets/69738699-3707-4a23-a8dd-8c69658890d6)

2. Look inside the POST requests in the HTTP history. What is the password for the *snowballelf* account? **c4rrotn0s3**.
![image](https://github.com/user-attachments/assets/17563a9f-d24f-447c-a207-7785ed3067f0)

3. Use the credentials for any of the elves to authenticate to the Gift Scheduler website. What is the flag shown on the elves’ scheduling page? **THM{AoC-3lf0nth3Sh3lf}**.
4. What is the password for Marta May Ware’s account? **H0llyJ0llySOCMAS!**
5. Mayor Malware finally succeeded in his evil intent: with Marta May Ware’s username and password, he can finally access the administrative console for the Gift Scheduler. G-Day is cancelled! What is the flag shown on the admin page? **THM{AoC-h0wt0ru1nG1ftD4y}**.
