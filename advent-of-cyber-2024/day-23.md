# Day 23: You wanna know what happens to your hashes?

**Learning Objectives**

By finishing today’s task, you will learn about:
- Hash functions and hash values
- Saving hashed passwords
- Cracking hashes
- Finding the password of a password-protected document

### Hashed Passwords
A long time ago, before security was a “thing”, passwords were stored in cleartext along with the associated username. When the user tries to log in, the system compares the provided password for this account with the saved one. Consequently, if a user forgets their password, someone with enough access privileges can look at the table and respond with something like, “The password for `joebloggs` is `ASDF1234`.” This was a terrible idea, especially since a database can be stolen and its content leaked online. Unfortunately, users tend to use the same password for different services. Consequently, if an adversary discovers Joe Bloggs’s password from another service, they will try it on Joe’s other accounts, such as email.

To protect passwords, even in the case of a data breach, companies started to save a hashed version of the password. For that, we need to use a hash function. A hash function takes an input of any size and returns a fixed size value. For example, SHA256 (Secure Hash Algorithm 256) creates a 256-bit hash value. In other words, `sha256sum FILE_NAME` will return a 256-bit hash value regardless of whether the input file is a few bytes or several gigabytes.

Therefore, instead of saving the password `ASDF1234` verbatim, its *hash* is saved. For instance, if MD5 (Message Digest 5) is being used, then `ce1bccda287f1d9e6d80dbd4cb6beb60` would be saved. Problem solved? Not really. Firstly, MD5 is now considered insecure. Secondly, in addition to choosing a secure hash function, we should add a **salt**, i.e., _a random string of characters_, to the password before hashing it. In other words, instead of saving `hash(password)` in the table, we save `hash(password + salt)` along with the salt. Consequently, when the user tries to log in, the authentication system takes their password along with the saved salt, calculates its hash and compares it with the saved hash value; if identical, they are granted access. This makes the saved passwords more immune to a variety attacks.

Although it is recommended to use a modern secure hashing algorithm to calculate the hash value of the password concatenated with a random salt before saving it, reality is not that shiny. In many cases, there are issues in the implementation, be it due to negligence or ignorance. In a recent story, a social media platform was discovered to have saved 600 million passwords in plaintext for seven years, despite all the security guidelines warning against that. In other words, password cracking is not over yet.

### Password-Protected Files
On [Day 14](https://github.com/securechell/tryhackme-labs/blob/main/advent-of-cyber-2024/day-14.md), we saw how Mayor Malware intercepted network data to eavesdrop on the village. Technically speaking, he was attacking the confidentiality and integrity of **data in transit**. Today, we will explore how to view his password-protected document. Technically speaking, we will be attacking the confidentiality of the **data at rest**.

One aspect of our security requires us to protect data while it is stored on any storage device; examples include a flash memory drive, smartphone storage, laptop storage, and external drives. If an adversary gains access to any such device, we don’t want them to be able to access our files. Protecting data at rest is usually achieved by encrypting the whole disk or specific files on the disk.

On the other hand, encrypted storage and files can pose an obstacle for the good guys who are investigating a criminal case. Digital forensic investigators need to find a way to access the plaintext files to prove or disprove any wrongdoing. In this case, for his private investigation to succeed, Glitch must figure out how to access the encrypted PDF file on the disposed-off tablet. Glitch needs to play an offensive security role to break the security of the protected document.

### Passwords
Opening a password-protected document is impossible unless we know or can find the password. The problem is that many users prefer to pick relatively easy passwords that they can remember easily and then use the same password across multiple places. Have you ever wondered which passwords are most commonly used?

![image](https://github.com/user-attachments/assets/88fae2bb-1145-4f80-a8d7-d117b2d16c6b)

Of course, users might get a little bit creative and might replace a character with a symbol. They might append the current year, a memorable date, or a few random characters or numbers to the original word.

### Demonstration
It is time to crack some passwords. We will cover the following:

- Cracking a password found in a breached database
- Finding the password of an encrypted PDF

#### Data Breach and Hash Values
Mayor Malware had an online account in a now-defunct forum that was breached, and all its user data was leaked. After checking online, we were able to retrieve the Mayor’s password in hashed format. It is listed below.
Username: `mayor@email.thm`
Password Hash: `d956a72c83a895cb767bb5be8dba791395021dcece002b689cf3b5bf5aaa20ac`

We want to discover the original password. The first step in our approach is to figure out the type of the hash function. Then, we will try to hash different passwords from a password list until we find a match.

We have saved the above hash value in the `/home/user/AOC2024/hash1.txt` file for your convenience.
1. First, we will go to the `AOC2024` directory and then display the content of `hash1.txt`.
2. Copy the displayed hash. Selecting the text in the split view will copy it for you.
3. Next, we start one tool that helps identify hashes by issuing the command `python hash-id.py`.
4. Paste the copied hash. Right-clicking with your mouse will paste the copied text in split view.
5. Finally, we quit the tool using `CTRL`+`C`.

![image](https://github.com/user-attachments/assets/a8997e10-df89-4d04-b44b-59eb1b593bda)

![image](https://github.com/user-attachments/assets/9387fa7d-670c-468f-b173-c1d7aa345800)

6. Next, we will try passwords from `rockyou.txt`, a popular password wordlist from a real data breach. The command is as follows:
`john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash1.txt`
- `john` starts John the Ripper; the jumbo edition is installed on the machine
- `--format=raw-sha256` specifies the hash format, which we have figured out earlier that it is most likely a SHA-256
- `--wordlist=/usr/share/wordlists/rockyou.txt` sets the wordlist that we will use
- `hash1.txt` is the text file containing the hash value we are trying to crack

In our first attempt, `john` calculated the SHA-256 hash value for every password in `rockyou.txt` and compared it with the hash value in `hash1.txt`. Unfortunately, no password was found:

![image](https://github.com/user-attachments/assets/e85471c9-01da-4d74-8065-de8f02fc4dbd)

There is a high chance that Mayor Malware has made some transformation to his password. For example, he might have replaced `a` with `4` or added a couple of digits to his password. John can start from a long password list and attempt various common derivations from each of the passwords to increase its chances of success. This behaviour can be triggered through the use of **rules**. Various rules come bundled with John the Ripper’s configuration files; one is suited for lengthy wordlists, `--rules=wordlist`.

7. Adding the option `--rules=wordlist` to your `john` command line generates multiple passwords from each one. For instance, it appends and prepends single digits. It does various common substitutions; for example, `a` can be replaced with `@`, `i` can be replaced with `!`, and `s` can be replaced with `$`. Many more mutations and transformations are part of these rules. You can check all the underlying rules by checking the `[List.Rules:Wordlist]` section in `/etc/john/john.conf`, John’s configuration file. Unlike the first attempt, using John with this option should crack the hash for you:
`john --format=raw-sha256 --rules=wordlist --wordlist=/usr/share/wordlists/rockyou.txt hash1.txt`

![image](https://github.com/user-attachments/assets/2d317650-7cac-47e5-97fb-5fada7ae21ac)

We should note that `john` will not spend computing resources to crack an already-cracked password hash. Consequently, if you repeat a command that has successfully found a password earlier, you will get a message like “No password hashes left to crack (see FAQ)”. Let’s say that you executed the command listed above and you recovered the password; then, the next time you want to see that password, you would use `john` with the `--show` option, for example, `john --format=raw-sha256 --show hash1.txt`.

#### Opening an Encrypted PDF
Glitch has discovered Mayor Malware’s password used on the breached online forum. Although there is a high chance that this password will be used to access other online accounts created by the Mayor, Glitch does not want to go that route as it would violate the local laws and regulations. Instead of attempting anything illegal, he focused on the data he discovered in the Mayor’s trash. There is one interesting-looking PDF file that happens to be password-protected. You can help Glitch break it.

1. The first thing you need to do is to convert the password-protected file into a format that `john` can attack. Luckily, John the Ripper jumbo edition comes with the necessary tools. The different tools follow the naming style “format2john”. The terminal below shows a few examples.

![image](https://github.com/user-attachments/assets/ec25c5cb-8aea-4020-8ffd-aa991b9c2280)

2. You are interested in a *password-protected PDF*; therefore, `pdf2john.pl` should do the job perfectly for you. In the terminal below, you can see how to create a hash challenge from a PDF file. This hash value can later be fed to `john` to crack it.

![image](https://github.com/user-attachments/assets/f031cbf6-a30e-4e6b-8f29-59a6cd9a8030)

3. Enter the command: `john --rules=single --wordlist=wordlist.txt pdf.hash`
	- `--rules=single` covers more modification rules and transformations on the wordlist
	- `--wordlist=wordlist.txt` is the custom and personalized wordlist that we created
	- `pdf.hash` is the hash generated from the password-protected document

![image](https://github.com/user-attachments/assets/909f57f4-5ce5-4b42-88fa-2680245baaf7)

We can see we've cracked the password for the PDF: `M4y0rM41w4r3`.

Now, you have gained all the necessary knowledge to tackle the questions below and uncover what Mayor Malware has been hiding in his password-protected document.

### Answers
1. Crack the hash value stored in `hash1.txt`. What was the password? (Hint: Your command should be something like: john --format=HASH-FORMAT --rules=wordlist --wordlist=/usr/share/wordlists/rockyou.txt HASH_FILENAME). **fluffycat12**.
	- the command we used was: `john --format=raw-sha256 --rules=wordlist --wordlist=/usr/share/wordlists/rockyou.txt hash1.txt`

2. What is the flag at the top of the `private.pdf` file? (Hint: After you discover the password of the PDF file, use the following command to convert it to a text file so you can read it on the console: pdftotext private.pdf -upw PASSWORD Afterwards, you can use the following command to display the top of the file: head private.txt). **THM{do_not_GET_CAUGHT}**.
	- We know the password of the PDF (`M4y0rM41w4r3`). Now we need to unlock the PDF using the password. We will also convert it to text so it's more readable. Enter the command `pdftotext private.pdf -upw M4y0rM41w4r3`

![image](https://github.com/user-attachments/assets/a51fd024-7338-48d2-80f9-964640088619)

	- We can see `private.txt` has been created. 
	- Now enter the command `head private.txt`. We see the flag.

![image](https://github.com/user-attachments/assets/b43171dd-1f8f-4472-8378-3cabcf01685d)
