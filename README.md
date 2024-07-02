# LazyAdmin
This is a tutorial for the TryHackMe LazyAdmin challenge. A link to this challenge is here: https://tryhackme.com/r/room/lazyadmin
## Tools
Below are the primary tools I used for completing this challenge:
* Nmap
* Gobuster
* Crackstation.net
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
  
6. Let's try to further enumerate directories using \<target IP\>/content as our url: `gobuster dir -u 10.10.211.117/content -w /usr/share/wordlists/dirb/common.txt`
    * Now we've found some additional directories that we can explore.
      
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/9b742d67-b0ad-49bf-b5d3-2f2da1ca8ea9)

7. Explore the directories to see if we can find anything interesting.
8. /content/inc looks interesting. It lists a bunch of different files and folders that we can explore further. The files lastest.txt and the folder mysql_backup/ look particularly intriguing.
   
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/4c4837d4-fe55-4f61-9dfa-c838a04b6d4c)

10. Open lastest.txt, and you'll see that it contains the text "1.5.1", which appears to confirm that SweetRice 1.5.1 is, in fact, the version this site uses.
11. Next, click on the folder mysql_backup/
12. This folder contains a file mysql_bakup_20191129023059-1.5.1.sql. Download this file and open it in a text editor to see what information it contains.
13. Towards the bottom of the file, you'll notice there are some SQL commands that contain a username and password:

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/a817f1bb-6952-4a27-bed8-759123224766)

    * This isn't the actual password, but appears to be a hashed value of a password, so we'll need to crack it in order to find the actual password.

14. We'll use [crackstation.net](https://crackstation.net/) to crack it.
    * Navigate to the Crackstation website.
    * Paste the hash value into the box.
    * Then confirm you're not a robot and click "Crack Hashes."
   
![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/4d27c315-f975-4d19-9f69-3dcc0238f479)

    * Crackstation shows that it's an MD5 hash and cracks it successfully.

15. Now have have a username (manager), and a password that we cracked using crackstation.
16. Looking through the other directories we enumerate earlier, **/content/as** is a login page. Enter the credentials there.

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/df89be5e-d8a6-4908-9d20-476e61d132c4)

17. And we're in!

![image](https://github.com/fatmagician/LazyAdmin/assets/51951855/b24300c4-d8d3-4130-94c4-15488e5323ab)











