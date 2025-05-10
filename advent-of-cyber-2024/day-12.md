# Day 12: If I can’t steal their money, I’ll steal their joy!

**Learning Objectives**
- Understand the concept of race condition vulnerabilities
- Identify the gaps introduced by HTTP2
- Exploit race conditions in a controlled environment
- Learn how to fix the race

### Web Timing and Race Conditions
Conventional web applications are relatively easy to understand, identify, and exploit. If there is an issue in the code of the web application, we can force the web application to perform an unintended action by sending specific inputs. These are easy to understand because there is usually a direct relationship between the input and output. We get bad output when we send bad data, indicating a vulnerability. But what if we can find vulnerabilities using only good data? What if it isn't about the data but how we send it? This is where web timing and race condition attacks come into play! Let's dive into this crazy world and often hidden attack surface!   

In its simplest form, a web timing attack means we glean information from a web application by reviewing how long it takes to process our request. By making tiny changes in what we send or how we send it and observing the response time, we can access information we are not authorised to have.

Race conditions are a subset of web timing attacks that are even more special. With a race condition attack, we are no longer simply looking to gain access to information but can cause the web application to perform unintended actions on our behalf.  

Web timing vulnerabilities can be incredibly subtle. Based on the following [research](https://portswigger.net/research/listen-to-the-whispers-web-timing-attacks-that-actually-work), response time differences ranging from 1300ms to 5ns have been used to stage attacks. Because of their subtle nature, they can also be hard to detect and often require a wide range of testing techniques. However, with the increase in adoption of HTTP/2, they have become a bit easier to find and exploit.

### Typical Timing Attacks
Timing attacks can often be divided into two main categories:

- **Information Disclosures**
Leveraging the differences in response delays, a threat actor can uncover information they should not have access to. For example, timing differences can be used to enumerate the usernames of an application, making it easier to stage a password-guessing attack and gain access to accounts.

- **Race Conditions**
Race conditions are similar to business logic flaws in that a threat actor can cause the application to perform unintended actions. However, the issue's root cause is how the web application processes requests, making it possible to cause the race condition. For example, if we send the same coupon request several times simultaneously, it might be possible to apply it more than once.

This task focuses on **race conditions**.

### Intercepting the Request
We need to configure the environment so that, as a pentester, all web traffic from our browser is routed through Burp Suite, a powerful web vulnerability scanner. This allows us to see and manipulate the requests as we browse.

1. Set up Burp Suite as [here](https://github.com/securechell/tryhackme-labs/blob/main/advent-of-cyber-2024/day-5.md#practical).

### Application Scanning
As a penetration tester, one key step in identifying race conditions is to validate functions involving multiple transactions or operations that interact with shared resources, such as transferring funds between accounts, reading and writing to a database, updating balances inconsistently, etc.

1. Log in to the Warville banking application using the credentials: **Account no = 110, Password = tester**. Once logged in, you will see a dashboard.

### Verifying the Fund Transfer Functionality
We will perform a sample transaction inside the browser. This will generate multiple GET and POST requests, and whatever request we make will be passed through  Burp Suite.

1. Our current balance is **$1000**. We will send **$500** to another bank account with the **account number 111**, and while doing that, all our requests will be captured in Burp Suite.
2. Click the **Transfer** button. You will see the following message indicating that the amount has been transferred:
![image](https://github.com/user-attachments/assets/a2fcc628-27b5-4c4d-adcd-9b206fbbb43c)

3. Let's review the fund transfer HTTP POST request logged in the Burp Suite's HTTP history option under the Proxy tab.
![image](https://github.com/user-attachments/assets/e90b7864-f1ba-4f52-99ec-4fc2cbc65f17)

This picture shows that the `/transfer` endpoint accepts a POST request with parameters `account_number` and `amount`. The Burp Suite tool has a feature known as **Repeater** that allows you to send multiple HTTP requests. We will use this feature to duplicate our HTTP POST request and send it multiple times to exploit the race condition vulnerability.

4. Right-click on the POST request and click on Send to Repeater.
5. Navigate to the Repeater tab, where you will find the POST request that needs to be triggered multiple times. We can change the `account_number`, from `111`, and the `amount` value from `500` to any other value in the request as well, as shown below:
![image](https://github.com/user-attachments/assets/a81aa03b-28da-428e-9b17-3e6031be79f0)

6. Place the mouse cursor inside the request inside the Repeater tab in Burp Suite and press **Ctrl+R** to duplicate the tab. Press Ctrl+R ten times to have 10 duplicate requests ready for testing.
![image](https://github.com/user-attachments/assets/6e836d1c-4b81-4696-9979-0f5ce70ab907)

Now that we have 10 requests ready, we want to send them simultaneously. While one option is to manually click the Send button in each tab individually, we aim to send them all in parallel.

7. Click the **+** icon next to Request #10 and select Create tab group. This will allow us to group all the requests together for easier management and execution in parallel.
8. A dialogue box will appear asking you to name the group and select the requests to include. Name the group "funds", select all the requests, and click Create.
![image](https://github.com/user-attachments/assets/c3ed3caa-55f5-4e3d-9de0-d37f5a91936b)

Now, we are ready to launch multiple copies of our HTTP POST requests simultaneously to exploit the race condition vulnerability.

9. Select **Send group in parallel (last-byte sync)** in the dropdown next to the Send button. Once selected, the Send button will change to **Send group (parallel)**. Click to send all the duplicated requests in our tab group at the same time.
10. Once all the requests have been sent, navigate to the Tester account in the browser and check the current balance. You will notice that the tester's balance is negative because we successfully transferred more funds than were available in the account, exploiting the race condition vulnerability.
![image](https://github.com/user-attachments/assets/fcb3a861-39d3-47a2-be49-b0537cfedc00)

By duplicating ten requests and sending them in parallel, we are instructing the system to make ten simultaneous requests, each deducting $500 from the Tester account and sending it to account 111. In a correctly implemented system, the application should have processed the first request, locked the database, and processed the remaining requests individually. However, due to the race condition, the application handles these requests abruptly, resulting in a negative balance in the tester account and an inflated balance in account 111.


*Now that you understand the vulnerability, can you assist Glitch in validating it using the account number: 101 and password: glitch? Attempt to exploit the vulnerability by transferring over **$2000** from his account to the account number: 111.*  

### Fixing the Race
The developer did not properly handle concurrent requests in the bank's application, leading to a race condition vulnerability during fund transfers. When multiple requests were sent in parallel, each deducting and transferring funds, the application processed them simultaneously without ensuring proper synchronisation. This resulted in inconsistent account balances, such as negative balances in the sender’s account and excess funds in the recipient’s account. Here are some of the preventive measures to fix the race.

- **Use Atomic Transactions**: The developer should have implemented atomic database transactions to ensure that all steps of a fund transfer (deducting and crediting balances) are performed as a single unit. This would ensure that either all steps of the transaction succeed or none do, preventing partial updates that could lead to an inconsistent state.
- **Implement Mutex Locks**: By using Mutex Locks, the developer could have ensured that only one thread accesses the shared resource (such as the account balance) at a time. This would prevent multiple requests from interfering with each other during concurrent transactions.
- **Apply Rate Limits**: The developer should have implemented rate limiting on critical functions like funds transfers and withdrawals. This would limit the number of requests processed within a specific time frame, reducing the risk of abuse through rapid, repeated requests.

### Answers
1. What is the flag value after transferring over $2000 from Glitch's account? **THM{WON_THE_RACE_007}**.
- In browser sign in with credentials (account number: 101 and password: glitch).
- Transfer $1000 to account 111.
- Here's the request in Burp Suite:
![image](https://github.com/user-attachments/assets/a1b37724-be81-4b55-ab2e-d6fef47f2509)

- Send the request to Repeater.
- In Repeater, CTRL+R to make 6 requests.
- Add the tabs to a new group.
- Go to the dropdown by the Send button and select Send group in parallel (last-byte sync).
- Click Send group (parallel).
- Go back to Glitch's account in browser:
![image](https://github.com/user-attachments/assets/fcfbb86a-7089-42ec-99cb-6097829e0d2a)
