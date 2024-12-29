# Day 5: SOC-mas XX-what-ee?

**Learning Objectives**
- Understand the basic concepts related to XML
- Explore XML External Entity (XXE) and its components
- Learn how to exploit the vulnerability
- Understand remediation measures

### Important Concepts

#### **Extensible Markup Language (XML)**
XML is a commonly used method to transport and store data in a structured format that humans and machines can easily understand. Consider a scenario where two computers need to communicate and share data. Both devices need to agree on a common format for exchanging information. This agreement (format) is known as **XML**. You can think of XML as a digital filing cabinet.

![image](https://github.com/user-attachments/assets/6017ea4e-7b17-407c-af5d-ca9733898b14)

Just as a filing cabinet has folders with labelled documents inside, XML uses "tags" to label and organise information. These tags are like folders that define the type of data stored. This is what an XML looks like, a simple piece of text information organised in a structured manner:

![image](https://github.com/user-attachments/assets/0432a7f8-761c-4a9b-85dd-cf5e23680342)

In this case, the tags `<people>`, `<name>`, `<address>`, etc are like folders in a filing cabinet, but now they store data about Glitch. The content inside the tags, like "`Glitch`," "`Wareville`," and "`111000`" represents the **actual data** being stored. Like before, the key benefit of XML is that it is easily shareable and customisable, allowing you to create your own tags.

>**Definition**:
>XML is a markup language that defines a set of rules for encoding documents in a format that is both human-readable and machine-readable.

#### **Document Type Definition (DTD)**
Now that the two computers have agreed to share data in a common format, what about the structure of the format? Here is when the DTD comes into play. A DTD is a set of **rules** that defines the structure of an XML document. Just like a database scheme, it acts like a blueprint, telling you what elements (tags) and attributes are allowed in the XML file. Think of it as a guideline that ensures the XML document follows a specific structure.

For example, if we want to ensure that an XML document about `people` will always include a `name`, `address`, `email`, and `phone`, we would define those rules through a DTD as shown below:

![image](https://github.com/user-attachments/assets/a5d09494-dbfc-45ac-8451-76bf886b97f0)

In the above DTD, `<!ELEMENT>`  defines the elements (tags) that are allowed, like name, address, email, and phone, whereas `#PCDATA` stands for parsed `people` data, meaning it will consist of just plain text.

>**Definition**:
>A DTD is a set of **rules** that defines the structure of an XML document.

#### **Entities**
So far, both computers have agreed on the format, the structure of data, and the type of data they will share. Entities in XML are placeholders that allow the insertion of large chunks of data or referencing internal or external files. They assist in making the XML file easy to manage, especially when the same data is repeated multiple times. Entities can be defined internally within the XML document or externally, referencing data from an outside source. 

For example, an external entity references data from an external file or resource. In the following code, the entity `&ext;` could refer to an external file located at "`http://tryhackme.com/robots.txt`", which would be loaded into the XML, if allowed by the system:

![image](https://github.com/user-attachments/assets/93880e47-c7f4-4896-a1b8-0ef69739c084)

We are specifically discussing **external entities** because it is one of the main reasons that XXE is introduced if it is not properly managed.

#### **XML External Entity (XXE)**

After understanding XML and how entities work, we can now explore the XXE vulnerability. XXE is an attack that takes advantage of **how XML parsers handle external entities**. When a web application processes an XML file that contains an external entity, the parser attempts to load or execute whatever resource the entity points to. If necessary sanitisation is not in place, the attacker may point the entity to any malicious source/code causing the undesired behaviour of the web app.

For example, if a vulnerable XML parser processes this external entity definition:

![image](https://github.com/user-attachments/assets/3c0f22da-435a-45db-a339-ff824f4d5ec5)

Here, the entity `&thmFile;` refers to the sensitive file `/etc/passwd` on a system. When the XML is processed, the parser will try to load and display the contents of that file, exposing sensitive information to the attacker.  

In the upcoming tasks, we will examine how XXE works and how to exploit it.

>**Definition**:
>XML External Entity injection (XXE) is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. It often allows an attacker to view files on the application server filesystem, and to interact with any back-end or external systems that the application itself can access.

### Practical 

We will analyse an application that allows users to view and add products to their carts and perform the checkout activity. You can access the Wareville application hosted on `http://10.10.93.248`. This application allows users to request their Christmas wishes.

**Flow of the Application**

As a penetration tester, it is important to first analyse the flow of the application. First, the user will browse through the products and add items of interest to their wishlist at `http://10.10.93.248/product.php`.
1. Click on the *Add to Wishlist* under *Wareville's Jolly Cap*.
2. After adding products to the wishlist, click the *Cart* button or visit `http://10.10.93.248/cart.php` to see the products added to the cart. On the Cart page, click *Proceed to Checkout* to buy the items.
3. On the checkout page, the user will be prompted to enter his name and address. Enter any name and address, and click *Complete Checkout* to place the wish. Once you complete the wish, you will be shown the message **"Wish successful. Your wish has been saved as Wish #21"**.

![image](https://github.com/user-attachments/assets/a0bb95e0-06cb-401a-8e1e-c44315d2b46b)

**Wish #21** indicates the wishes placed by a user on the website. Once you click on **Wish #21**, you will see a forbidden page because the details are only accessible to admins. But can we try to bypass this and access other people's wishes? This is what we will try to perform in this task.

**Intercepting the Request**

Before discussing exploiting XXE on the web, let's learn how to intercept the request. First, we need to configure the environment so that, as a pentester, all web traffic from our browser is routed through Burp Suite. This allows us to see and manipulate the requests as we browse. 

We will use Burp Suite, a powerful web vulnerability scanner, to intercept and modify requests for this exploitation.

1. On the desktop of the AttackBox, you will see a Burp Suite icon. Once you click the icon, Burp Suite will open with an introductory screen. Click on the Next button.
2. On the next screen, you will have the option to Start Burp. Click *Start Burp* to start the tool. Once Burp Suite has started, you will see its main interface with different tabs, such as *Proxy*, *Intruder*, *Repeater* and others.

![image](https://github.com/user-attachments/assets/e199a643-9d7d-469a-9318-784a242d5d2f)

3. Inside Burp Suite, click the Settings tab at the top right. You will see Burp's browser option available under the Tools section. Enable *Allow Burp's browser to run without a sandbox* option and click on the **Close** icon on the top right corner of the Settings tab as shown below:

![image](https://github.com/user-attachments/assets/88f58a58-a060-4329-9a3e-d262ee12f3c6)

4. After allowing the browser to run without a sandbox, we would now be able to start the browser with pre-configured Burp Suite's proxy. Navigate to the *Open browser* option located at Proxy tab -> Intercept.  
5. Turn Intercept off.

![image](https://github.com/user-attachments/assets/d5bfbe0f-dae5-4f45-9817-2bf995d3513a)

Click *Open browser* and go to `http://10.10.93.248`.
 
6. Go back to the browser and explore the URL. All the requests are intercepted and can be seen under Proxy -> HTTP history.

![image](https://github.com/user-attachments/assets/6f1c36eb-9400-45ce-a70f-5e25a62a4f2b)


**What is Happening in the Backend?**

Now, when you visit the URL, `http://10.10.93.248/product.php`, and click *Add to Wishlist,* an AJAX call is made to `wishlist.php` with the following XML as input.

![image](https://github.com/user-attachments/assets/d9cc3829-382e-4291-8cb0-95628f060f1b)

In the above XML, **<product_id>** tag contains the ID of the product, which is **1** in this case. Now, let's review the Add to Wishlist request logged in Burp Suite's *HTTP History* option under the *Proxy* tab. As discussed above, the request contains XML being forwarded as a POST request, as shown below:

![image](https://github.com/user-attachments/assets/a86aa517-132f-402f-ae4f-6fb985c12348)

![image](https://github.com/user-attachments/assets/3b93e094-23f3-4fb6-93af-cf47bc1636cc)

This `wishlist.php` accepts the request and parses the request using the following code:

![image](https://github.com/user-attachments/assets/559eba29-16b4-4be6-beb2-ee1fa73955c7)

**Preparing the Payload**

When a user sends specially crafted XML data to the application, the line `libxml_disable_entity_loader(false)` allows the XML parser to load external entities. This means the XML input can include external file references or requests to remote servers. When the XML is processed by `simplexml_load_string` with the `LIBXML_NOENT` option, the web app resolves external entities, allowing attackers access to sensitive files or allowing them to make unintended requests from the server.  

What if we update the XML request to include references for external entities? We will use the following XML instead of the above XML:

![image](https://github.com/user-attachments/assets/dfb715ca-5f44-4f02-8a59-79ca368eb3e6)

When we send this updated XML payload, the first two lines introduce an external entity called **payload**. The line <**!ENTITY payload SYSTEM "/etc/hosts">** tells the XML parser to replace the **&payload;** reference with the contents of the file **/etc/hosts** on the server. When the XML is processed, instead of a normal **product_id**, the application will try to load and include the contents of the file specified in the entity (`/etc/hosts`).

**Exploitation**

Now, let's perform the exploitation by repeating the request we captured earlier. The Burp Suite tool has a feature known as Repeater that allows you to send multiple HTTP requests. We will use this feature to duplicate our HTTP POST request and send it multiple times to exploit the vulnerability.
1. Right-click on the `wishlist.php` POST request and click on Send to Repeater.
2. Now, switch to the Repeater tab, where you'll find the POST request that needs to be modified. We will update the XML payload with the new data as shown below and then Send the modified request.

![image](https://github.com/user-attachments/assets/b14911ef-2996-4379-ba61-66f895ece548)

3. When we clicked Send, the server processed the malicious XML payload, which included the external entity reference to `/etc/hosts`. As a result, the `wishlist.php` responded with the contents of the `/etc/hosts` file, leading to an XXE vulnerability.

![image](https://github.com/user-attachments/assets/8c5a37ad-6a69-4db2-a0b4-299366015966)


### Time for Some Action

Now that you've identified a vulnerability in the application, it's time to see it in action! McSkidy Software has tasked us with finding loopholes — let's take it a step further and assess the potential impact this vulnerability could have on the application.

Earlier, we discovered a page accessible only by admins, which seems like an exciting target. What if we could use the vulnerability we've found to access sensitive information, like the wishes placed by the townspeople?

Now that our objective is clear, let's leverage the vulnerability we discovered to read the contents of each wishes page and demonstrate the full extent of this flaw to help McSkidy secure the platform.
1. To get started, let's recall the page that is only accessible by admins: `/wishes/wish_21.txt`. Using this path, we just need to guess the potential absolute path of the file. *Typically*, web applications are hosted on `/var/www/html`. With that in mind, let's build our new payload to read the wishes while leveraging the vulnerability.
2. In Repeater, replace `etc/hosts` with `/var/www/html/wishes/wish_21.txt`. Click Send .

![image](https://github.com/user-attachments/assets/45fe58af-d755-4d09-a512-15b8846b4588)

3. The Wish details are now visible:
   
![image](https://github.com/user-attachments/assets/7c61ceec-2178-446e-8649-4fea7d697541)

Try with some other wish numbers:

![image](https://github.com/user-attachments/assets/73c78c94-aa72-4316-aa82-132ac4627ffa)

![image](https://github.com/user-attachments/assets/0894206f-0556-4e00-8e76-9b31a2d07f74)

After iterating through the wishes, we have proved the potential impact of the vulnerability, and anyone who leverages this could read the wishes submitted by the townspeople of Wareville.

### Conclusion

It was confirmed that the application was vulnerable, and the developers were not at fault since they only wanted to give the townspeople something before Christmas. However, it became evident that bypassing security testing led to an application that did not securely handle incoming requests.

As soon as the vulnerability was discovered, McSkidy promptly coordinated with the developers to implement the necessary mitigations. The following proactive approach helped to address the potential risks against XXE attacks:

- **Disable External Entity Loading**: The primary fix is to disable external entity loading in your XML parser. In PHP, for example, you can prevent XXE by setting `libxml_disable_entity_loader(true)` before processing the XML.
- **Validate and Sanitise User Input**: Always validate and sanitise the XML input received from users. This ensures that only expected data is processed, reducing the risk of malicious content being included in the request. For example, remove suspicious keywords like `/etc/host`, `/etc/passwd`, etc, from the request.

After discovering the vulnerability, McSkidy immediately remembered that a CHANGELOG file exists within the web application, stored at the following endpoint: `http://10.10.238.226/CHANGELOG`.

### Answers
1. What is the flag discovered after navigating through the wishes? **THM{Brut3f0rc1n6_mY_w4y}**.

We need to look through all of the wishes to look for sensitive information.
- Right-click the request and select Send to Intruder.
- In Intruder, highlight `1` (in `wish_1.txt`) - this will be our entry point.
  
![image](https://github.com/user-attachments/assets/17ca0443-f571-420d-86ce-dcb5ad723a5b)

- Click Add.
  
![image](https://github.com/user-attachments/assets/ed4d0e39-4c5f-4b71-bd68-b062093d1e48)

- Go to Payloads tab and change settings to:
	- Payload type = Numbers
	- From = 1
	- To = 20
	- Steps = 1
 
![image](https://github.com/user-attachments/assets/d66ebdc7-18ee-4bfe-9e82-76c8273b56e7)

- Click Start Attack.
- The attack goes through our (wish) numbers. Select the Response tab to see the contents of each wish.
  
![image](https://github.com/user-attachments/assets/56f0444d-883f-4a3b-9d12-d813f732754c)

- The flag is in Wish #15.

![image](https://github.com/user-attachments/assets/517be03f-b06f-4340-bf62-35bdceebaf94)

2. What is the flag seen on the possible proof of sabotage? **THM{m4y0r_m4lw4r3_b4ckd00rs}**.

- Type `http://10.10.238.226/CHANGELOG` in browser. Flag in text.
  
![image](https://github.com/user-attachments/assets/3a05d85b-2ffb-4171-850f-ab154be94a84)
