# Day 1: Linux CLI - Shells Bells

**Learning Objectives**
- Learn the basics of the Linux command-line interface (CLI)
- Explore its use for personal objectives and IT administration
- Apply your knowledge to unveil the Christmas mysteries

### Working With the Linux CLI
Linux has a powerful command-line interface, allowing you to use and manage the system simply by typing commands on your keyboard.
- Then type `ls` to list the contents of the current directory. This command will show you McSkidy's files.
- After that, type `cat README.txt` to display the file contents.


**Navigating the Filesystem**

You might have noticed the "Guides" directory when you ran `ls` last time - that's likely the directory we need. Your CLI journey began at McSkidy's home directory (you can verify this by running `pwd`), but now let's switch to the guides directory.
- Switch the directory by running `cd Guides`. You will appear at `/home/mcskidy/Guides`.
- Run the `ls` command again to list the content of the guides directory (it will be empty).

**Looking for the Hidden Guide**

In Linux, files and directories can be hidden from plain view if they start with a dot symbol (e.g., `.secret.txt`). Such a feature is often used by IT administrators to hide system files, by attackers to hide malware, and now by McSkidy to hide the precious guide from bad bunnies!

![image](https://github.com/user-attachments/assets/aece6d83-db86-44dc-98e0-f688b6cd0b14)

**Grepping the Logs**

In her guide, McSkidy refers to `/var/log/`, a Linux directory where all security events (logs) are stored. Indeed, every SOC analyst at TBFC will confirm that the best way to find evil bunnies is to check the logs. *Log files are usually very big, and looking through them with cat is not ideal.* Thus, let's use `grep`, a command to look for a specific text inside a file.

![image](https://github.com/user-attachments/assets/0cf2246e-ca2d-4b9c-9c4b-5ceee3e2f2f0)

**Finding the Files**

You can see a lot of failed logins on the "socmas" account, all from the HopSec location! They were clearly trying to break into SOC-mas, Wareville's Christmas ordering platform. What if bad bunnies left some malware there? Let's follow McSkidy's guide and look for Eggsploits and Eggshells with `find` - a command that searches for files with specific parameters, such as `-name`:
- Run `find /home/socmas -name *egg*` to search for "eggs" in the socmas home directory.
- **Note:** `find` is a powerful command. Check out its [documentation](https://man7.org/linux/man-pages/man1/find.1.html) for more details.

![image](https://github.com/user-attachments/assets/2b58cb80-5a3d-43e6-8227-7d54ea328679)

**Analyzing the Eggstrike**

Looks like you found something, `eggstrike.sh`! Files with the `.sh` extension contain CLI commands and are called [[Shell|shell scripts]]. Such scripts are used both by IT teams to automate things and by attackers to quickly run malicious commands. Let's display the suspicious script's content and try to understand it:

![image](https://github.com/user-attachments/assets/43f8860f-fea8-4f9a-8699-2d1e3f129cd9)

1. Lines starting with `#` are just *comments* and are not the actual commands.
2. `cat wishlist.txt | sort | uniq` lists unique items from the wishlist.txt.
3. The command then sends the output (unique orders) to the /tmp/dump.txt file.
4. `rm wishlist.txt` deletes the wishlist file (containing Christmas wishes).
5. `mv eastmas.txt wishlist.txt` replaces the original file with eastmas.txt.

**CLI Features**

The Eggstrike script you read seems to be stealing Christmas wishes and replacing them with the fake ones! You might have noticed that the commands in the script are a bit complex, but that's not unusual since the script author is no other than Sir Carrotbane, the leader of HopSec's red team. Let's explore the special symbols below:

![image](https://github.com/user-attachments/assets/d5229815-8601-447e-85b4-b4180ea99921)


### Sir Carrotbane Attacks

**System Utilities**

There are hundreds of CLI commands to view and manage your system; e.g.: `uptime` to see how much time your system is running, `ip addr` to check your IP address, and `ps aux` to list all processes. You may also check the usernames and hashed passwords of users, such as McSkidy, by running `cat /etc/shadow`. However, you'd need root permissions to do that.

**Root User**

Root is the default, ultimate Linux user who can do anything on the system. You can *switch the user to root* with `sudo su`, and return back to McSkidy with the `exit` command. Only root can open `/etc/shadow` and edit system settings, so this user is often a main target for attackers. If at any moment you want to verify your current user, just run `whoami`!

![image](https://github.com/user-attachments/assets/86d69838-2820-4943-8c53-31660bf990ca)

**Bash History**
Did you know that every command you run is saved in a **hidden** history file, also called Bash history? It is located at every user's home directory: `/home/mcskidy/.bash_history` for McSkidy, and `/root/.bash_history` for root, and you can check it with a convenient `history` command, or just read the files directly with `cat`. Let's check if Sir Carrotbane with his bad bunnies left their traces in history!

![image](https://github.com/user-attachments/assets/2db6015d-37b4-4660-9af3-bb8b5e8bc851)

![image](https://github.com/user-attachments/assets/68787cc5-e519-42fc-866a-1822ed175faf)


### Answers
1. Which CLI command would you use to list a directory? *la*
2. Which command helped you filter the logs for failed logins? *grep*
3. Which command would you run to switch to the root user? *sudo su*
4. What flag did Sir Carrotbane leave in the root bash history? *THM{until-we-meet-again}*
