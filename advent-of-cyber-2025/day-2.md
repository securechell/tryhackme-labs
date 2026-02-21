# Day 2: Phishing - Merry Clickmas

**Learning Objectives**
- Understand what social engineering is
- Learn the types of phishing
- Explore how red teams create fake login pages
- Use the Social-Engineer Toolkit to send a phishing email

### Social Engineering
Social engineering refers to manipulating a user to make a mistake. Examples include sharing a password, opening a malicious file, and approving a payment. The term “social” means that the target of such an attack is human beings, not computer systems. Consequently, the attacker relies on psychological tricks to get the target user to cooperate. Some psychological factors that can play a key role in the success of such attacks are urgency, curiosity, and authority. This is why some refer to social engineering as “human hacking”.

#### Phishing
Phishing is a subset of social engineering in which the communication medium is mostly messages. At one point, the most common phishing attacks happened via email; however, the spread of smartphones, along with ubiquitous Internet access, has spread phishing to short text messages (smishing), voice calls (vishing), QR codes (quishing), and social-media direct messages. The attacker’s purpose is to make the target user click, open, or reply to a message so that the attacker can steal information, money, or access.

### Building the Trap

In light of several recent cyber threats against The Best Festival Company (TBFC), the local red team has scheduled several penetration tests. Part of this is to ensure employees are diligent when clicking links and that the company is well protected against the latest phishing attacks. This type of authorised phishing is a proven way to learn whether cyber security awareness training has fruited.

In this task, you will be part of the TBFC local red team. You will help them plan and execute their phishing campaign. 

You must sound very convincing as a pentester for a successful phishing attack. It’s not only how you write the phishing email or messages, but also how you set up the trap for the target. The trap can be anything, depending on your objectives and the research you conduct on the target. Sometimes, attackers aim to compromise the target’s machine by attaching a malicious file to a phishing email. Sometimes attackers craft a web page that mimics a legitimate login page to steal the target’s credentials.

In this task, we aim to acquire the target user’s login credentials. Our trap would be a fake TBFC portal login page, which we attach to the phishing email and send to the target. But a login page itself is not enough. We need to host it and implement some logic to capture the credentials entered by the target.

To run the script, use `./server.py` and it will start listening for any credentials. If the target gets trapped and enters the credentials, it will be shown on the same terminal.

<img width="781" height="87" alt="image" src="https://github.com/user-attachments/assets/5e90bfd1-b284-4e31-b780-27d526a5034a" />

The above message indicates that the phishing web application is listening on port 8000; moreover, the `0.0.0.0` implies that it is bound to *all interfaces*. To confirm what the user will see, use Firefox on the AttackBox and browse to `http://10.81.117.94:8000` (AttackBox IP) or `http://127.0.0.1:8000`; either of these addresses will show you what the user will see.

<img width="922" height="745" alt="image" src="https://github.com/user-attachments/assets/3f07c6e5-a60d-49c8-a1d6-1ea8b9dc2dcb" />

### Delivery via Social-Engineer Toolkit (SET)

As our phishing page is ready, we can now prepare and send the phishing email to our target users. Sending it from our personal email is the worst idea. Ideally, the email should appear to be coming from a legitimate-looking sender; eg, we can pretend to be somebody the target user trusts or expects to get such an email from. The more a phishing email appears realistic, the more likely it is for the target user to believe it and get phished. The question is: how we can send a realistic-looking email that contains our fake login page?

One solution is to use the [Social-Engineer Toolkit (SET)](https://github.com/trustedsec/social-engineer-toolkit), an open-source tool primarily designed by David Kennedy for social engineering attacks. It offers a wide range of features. In particular, it lets you compose and send a phishing email. In the current scenario, we will use this tool to create and send a phishing email to the target user. 

**Creating the phishing email through the SET tool** 
 - This will involve multiple steps, each asking different questions about the phishing email you intend to send. Be patient.
 - Open another tab in the terminal.
 - To start the tool, type `setoolkit` into the terminal. It will present a menu with multiple options. At the bottom, you will see `set>`, where you can input your desired option number. For our case, select option `1` **Social-Engineering Attacks**. 
 **Note:** If you choose the wrong option at any stage, the option `99` will take you back to the main menu to start again. However, if you commit any mistake while writing the phishing email, you would have to press `Ctrl + C` to return to the main menu. The social engineering attacks cover various attacks from spear-phishing and mass mailer attacks to wireless access point attacks.

<img width="597" height="436" alt="image" src="https://github.com/user-attachments/assets/90f32aa3-ed06-4dd1-b1a0-9790fae160d7" />

 - Choosing `1` will display another menu with the type of social engineering attack we want to use in our attack.

 <img width="667" height="457" alt="image" src="https://github.com/user-attachments/assets/c30a8fb8-fb85-478d-9aaf-ca727b066883" />

 - In this case, we'll type `5` **Mass Mailer Attack**
 - Now, we're be asked to select between two options. One allows us to send the phishing email to a single address, while the other enables us to send an email to many people.
 - Select option `1` **E-Mail Attack Single Email Address**

<img width="910" height="433" alt="image" src="https://github.com/user-attachments/assets/6eae6123-e818-44d3-8c8c-46d6d873c1fd" />

 - Now, we will have several questions to answer and various fields to fill out. The first set of questions concerns the email addresses and how the email will be routed and delivered. After each input provided, press **Enter** for the next question.
	- **Send email to**: Let’s target `factory@wareville.thm`
	- **How to deliver the email**: Choose `Use your own server or open relay`
	- **From address**: We know that the guys at the toy factory communicate regularly with Flying Deer, the shipping company, so use `updates@flyingdeer.thm` as the source email address
	- **From name**: `Flying Deer`
	- **Username for open-relay**: Leave blank, hit **Enter**
	- **Password for open-relay**: Leave it blank, hit **Enter**
	- **SMTP email server address**: We will deliver directly to the TBFC mail server by entering `10.81.160.249` (Target Machine IP)
	- **Port number for the SMTP server**: Leave the default value of `25`, don't type anything, just press **Enter**

 - The next set of questions will ask if you want to send it as a high priority or attach a file.
	- **Flag this message as high priority:** We will answer with `no`
	- **Do you want to attach a file:**  `n`
	- **Do you want to attach an inline file:** `n`

 - Finally, we pick an email subject and enter the message contents in plaintext or HTML.
	- **Email subject:** We need to think of something convincing, e.g. “Shipping Schedule Changes”
	- **Send the message as HTML or plain:** We will keep the default choice of plaintext, don't type anything, just press **Enter**
	- **Enter the body of the message:** Create and type any convincing message. Make sure to include the URL `http://10.81.117.94:8000` to check if the target will fall for this trick. Press **Enter** for each new line. Type `END` when finished. 

<img width="777" height="214" alt="image" src="https://github.com/user-attachments/assets/31b1c340-093e-4e9b-813e-0222488422b5" />

Now, the phishing email has been sent to the target.
 - Open the terminal where our `server.py` script is running to see if the user gets trapped and enters their credentials. **Note:** You may have to wait for 1 - 2 minutes and observe the terminal for any credentials entered by the user.

<img width="751" height="625" alt="image" src="https://github.com/user-attachments/assets/01a814e0-aed1-48f1-87df-cd08dd8da558" />

To the TBFC red team’s surprise, they received at least one set of working credentials. This result is alarming; it means that an adversary could succeed in a similar attack if it has not already been done. Considering the received credentials, if an adversary gains such access, they can easily wreck the whole gift delivery system. It is vital to check if such an attack has taken place and act accordingly.

### Answers
1. What is the password used to access the TBFC portal? *unranked-wisdom-anthem*
2. Browse to `http://10.81.160.249` from within the AttackBox and try to access the mailbox of the `factory` user to see if the previously harvested `admin` password has been reused on the email portal. What is the total number of toys expected for delivery? *1984000*
