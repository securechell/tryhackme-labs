# Day 13: It came without buffering! It came without lag!

**Learning Objectives**
- Learn about WebSockets and their vulnerabilities.
- Learn how WebSocket Message Manipulation can be done.

### Introduction to WebSocket
**WebSockets** let your browser and the server keep a constant line of communication open. Unlike the old-school method of asking for something, getting a response, and then hanging up, WebSockets are like keeping the phone line open so you can chat whenever you need to. Once that connection is set up, the client and server can talk back and forth without all the extra requests.

WebSockets are great for live chat apps, real-time games, or any live data feed where you want constant updates. After a quick handshake to get things started, both sides can send messages whenever. This means less overhead and faster communication when you need data flowing in real-time.

### Traditional HTTP Requests vs. WebSocket
When you use regular HTTP, your browser sends a request to the server, and the server responds, then closes the connection. If you need new data, you have to make another request. Think of it like knocking on someone's door every time you want something—they'll answer, but it can get tiring if you need updates constantly.

Take a chat app as an example. With HTTP, your browser would keep asking, "Any new messages?" every few seconds. This method, known as polling, works but isn’t efficient. Both the browser and the server end up doing a lot of unnecessary work just to stay updated.

WebSockets handle things differently. Once the connection is established, it remains open, allowing the server to push updates to you whenever there’s something new. It’s more like leaving the door open so updates can come in immediately without the constant back-and-forth. This approach is faster and uses fewer resources.

### WebSocket Vulnerabilities
While WebSockets can boost performance, they also come with security risks that developers need to monitor. Since WebSocket connections stay open and active, they can be taken advantage of if the proper security measures aren't in place. Here are some common vulnerabilities:

- **Weak Authentication and Authorisation:** Unlike regular HTTP, WebSockets don't have built-in ways to handle user authentication or session validation. If you don't set these controls up properly, attackers could slip in and get access to sensitive data or mess with the connection.
- **Message Tampering:** WebSockets let data flow back and forth constantly, which means attackers could intercept and change messages if encryption isn't used. This could allow them to inject harmful commands, perform actions they shouldn't, or mess with the sent data.
- **Cross-Site WebSocket Hijacking (CSWSH):** This happens when an attacker tricks a user's browser into opening a WebSocket connection to another site. If successful, the attacker might be able to hijack that connection or access data meant for the legitimate server.
- **Denial of Service (DoS):** Because WebSocket connections stay open, they can be targeted by DoS attacks. An attacker could flood the server with a ton of messages, potentially slowing it down or crashing it altogether.

### What Is WebSocket Message Manipulation?
WebSocket Message Manipulation is when an attacker intercepts and changes the messages sent between a web app and its server. Unlike regular HTTP requests that go back and forth one at a time, WebSockets keep a connection open, allowing constant two-way communication. This is what makes WebSockets great for real-time apps, but it also opens the door for attacks if proper security isn't in place.

In this type of attack, a hacker could intercept and tweak these WebSocket messages as they're being sent. Let's say the app is sending sensitive info, like transaction details or user commands—an attacker could change those messages to make the app behave differently. They could bypass security checks, send unauthorised requests, or alter key data like usernames, payment amounts, or access levels.

This kind of manipulation can also lead to more significant problems. Hackers could inject harmful code or try to get higher-level access. For instance, they might change a message to give themselves admin rights or insert malicious commands to take control of the server.

What makes this attack so dangerous is that WebSocket connections often don't have the same security protections as traditional HTTP connections, like End-to-End Encryption, which encrypts the request body of an HTTP request using JavaScript using an AES key or RSA public key stored in the JavaScript file. If developers don't add vigorous checks like message validation or encryption, it's easy for attackers to exploit these gaps. By tampering with the data being sent, attackers can cause all sorts of damage, from unauthorised actions to full system compromises.

The impact of changing WebSocket messages depends on how the app uses them and what kind of data is being sent. Here's a breakdown of what can happen:

- **Doing Things Without Permission:** If someone can tamper with WebSocket messages, they could impersonate another user and carry out unauthorised actions such as making purchases, transferring funds, or changing account settings. For example, if a WebSocket manages payment transactions, an attacker could manipulate the transaction amount or reroute the payment to their own account.
- **Gaining Extra Privileges:** Attackers could also manipulate messages to make the system think they have more privileges than they actually do. This could let them access admin controls, change user data, view sensitive info, or mess with system settings.
- **Messing Up Data:** One of the significant risks is data corruption. If someone is changing the messages, they could feed bad data into the system. This could mess with user accounts, transactions, or anything else the app handles. They could change things in real-time and disrupt everyone's work in circumstances such as a shared document or tool.
- **Crashing the System:** An attacker could also spam the server with bad requests, causing it to slow down or crash. If this happens enough, the system could go offline, causing serious downtime for users and businesses.

Without good security checks, this kind of message tampering can lead to anything from unauthorised actions to the downing of an entire service.

### Exploitation
1. Navigate to `http://10.10.253.44`.
2. Make sure Burp is set up on web browser. On Firefox, click icon in top right corner to turn Burp on:
![image](https://github.com/user-attachments/assets/4bc5a7ea-3254-445b-9016-56059d6eb973)

3. Open Burp Suite, navigate to Proxy > Intercept > Proxy Settings. Scroll down to WebSocket interception rules and ensure the settings below are turned on.
![image](https://github.com/user-attachments/assets/96fb39de-cf68-43c9-80b6-0831a7d19580)

4. Once done, close the window and turn Intercept on.
5. Go back to the browser and click Track.
6. Burp Proxy will intercept the WebSocket traffic, as shown below
![image](https://github.com/user-attachments/assets/201e250c-89a9-400f-bdd4-c9e56d175348)

![image](https://github.com/user-attachments/assets/c22ebf54-932f-458d-8431-52d5138a5d15)

7. Change the value of the "userId" parameter from **5** to **8** and click the Forward button.
![image](https://github.com/user-attachments/assets/abb7d858-d96b-4921-aa17-20805cb747ba)

8. Turn off Intercept, go to browser and check the Community Reports:
![image](https://github.com/user-attachments/assets/2aa50669-8c3b-41d1-9d41-0932832ff92f)

### Manipulating the Messaging
Following the successful identification of the WebSocket Message Manipulation vulnerability, Glitch continued testing for other ways to exploit the application. This time, he wanted to see if the messages posted on the app could be altered and manipulated. Is it possible to post using a different user ID?

1. Refresh browser.
2. In Burp turn Intercept on.
3. Type a message in chat box, and Send.
4. Return to Burp, change "sender" from 5 to 8, click Forward, the quickly turn Intercept off.
5. Go to Community Reports and:
![image](https://github.com/user-attachments/assets/9f2d00d4-59ad-4f34-940b-d6faf068b002)

### Answers
1. What is the value of Flag1? **THM{dude_where_is_my_car}**.
2. What is the value of Flag2? **THM{my_name_is_malware._mayor_malware}**.
