# TryHackMe - Year of the Rabbit Room
<img width="572" alt="image" src="https://user-images.githubusercontent.com/114961392/210897955-fa6f2b20-0568-4e9f-91d7-c14155d89a16.png">

## Reconnaissance
Let's start by scanning all ports with nmap then run a version and script scan on all open ports.  

<img width="485" alt="image" src="https://user-images.githubusercontent.com/114961392/210898885-3a3969ce-b879-4e3f-8101-837d04a0da16.png">

Ports 21 (FTP), 22 (SSH), and 80 (HTTP) are open. FTP does not have anonymous login so I will need to find the credentials if I want to connect.  
Let's check out the webpage on port 80.  

<img width="407" alt="image" src="https://user-images.githubusercontent.com/114961392/210899369-d7258103-4577-4cd2-bc5e-3a8697d761f5.png">

It is just the standard apache2 default page with nothing interesting in the source code.  
Now I will try brute forcing for any hidden directories.  

<img width="473" alt="image" src="https://user-images.githubusercontent.com/114961392/210899751-55c290e6-99b1-4003-8d89-41c6cc247a7e.png">

The only mildly interesting directory here is the **/assets** directory which has a rick roll video and a style sheet.  

<img width="248" alt="image" src="https://user-images.githubusercontent.com/114961392/210899847-7c6f4caa-b5d2-4f95-aae1-83d4a1aef326.png">

However in the stylesheet it reveals another hidden directory **/sup3r_s3cr3t_fl4g.php**.  

<img width="277" alt="image" src="https://user-images.githubusercontent.com/114961392/210900344-492fd0bf-d7a8-4dec-adee-b8fd90c10ddf.png">

Let's have a look at this directory. As soon as I navigate to the page I am presented with an alert telling me to turn off javascript. I will do this in Firefox's **about:config** settings then continue.  

<img width="256" alt="image" src="https://user-images.githubusercontent.com/114961392/210900567-7c9c2bdb-d776-4446-a108-f328f5a0867c.png">

After that I am presented with a new page with some text and a video. Unfortunetly the video does not load so I will have to work around it. The page source has nothing interesting so next I will check the requests in Burp Suite.  

<img width="634" alt="image" src="https://user-images.githubusercontent.com/114961392/210901432-e13b9f44-e1e9-4ea7-95e2-46720b165e7a.png">

Burp is showing an interesting request with a hidden directory. Let's have a look at this.  

<img width="344" alt="image" src="https://user-images.githubusercontent.com/114961392/210901571-1b0f904f-e3a2-4e2a-8b16-2f4b08a547e5.png">

This reveals a png image that we can download. I will copy this to my machine and investigate further.  
Exiftool, file, and steghide do not reveal anything interesting with the image however strings does.  

<img width="404" alt="image" src="https://user-images.githubusercontent.com/114961392/210901983-b278d189-8adb-4ca3-94bd-8d7962ee7ef6.png">

<img width="268" alt="image" src="https://user-images.githubusercontent.com/114961392/210902185-4e090a63-0580-40d2-8e9f-1d0696abb6d2.png">

## Exploitation
Strings reveals a username for FTP **(ftpuser)** and a list of passwords which only 1 is correct.  
I can use **hydra** to brute force these. First I will make a list of the passwords then I will use that with hydra.
**hydra -l ftpuser -P pass.txt 10.10.22.214 ftp**

<img width="831" alt="image" src="https://user-images.githubusercontent.com/114961392/210907106-e450e483-b847-42b0-a2fa-5fb6178c6f31.png">

This reveals the password for ftpuser.  
Now I can login to FTP with these credentials.  

<img width="830" alt="image" src="https://user-images.githubusercontent.com/114961392/210907381-78a6fc58-f452-45b8-849c-7b8f78bbb9e4.png">

Once I have logged into FTP, there is only 1 file in there so I will grab that and have a look.  

<img width="373" alt="image" src="https://user-images.githubusercontent.com/114961392/210907529-9baaa836-6e09-4be7-b64f-9797e666fd98.png">

The contents of the text file is encoded in the **brainfuck** language. I figured this out by copy and pasting some of the code into google and it gave me the results. Now I will use an online decoder to decode.  

<img width="586" alt="image" src="https://user-images.githubusercontent.com/114961392/210907837-8ebbeb58-4a52-4242-b60c-b28b6b04f005.png">

This reveals the username and password.  
Now we can use these credentials to connect through SSH.  

<img width="563" alt="image" src="https://user-images.githubusercontent.com/114961392/210908071-cee8942c-62b3-4c0a-be9a-85181462c156.png">

Once I am connected, I am presented with a login message saying:  
"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"  

Since there is no immediate flag once I connect, I will search for anything related to **s3cr3t** as the message suggests.  

<img width="463" alt="image" src="https://user-images.githubusercontent.com/114961392/210908430-8185afa5-c6ca-46c6-bcbd-b500ef622f51.png">

This reveals a password for user gwendoline. Now I can try to connect to their user account.  

<img width="266" alt="image" src="https://user-images.githubusercontent.com/114961392/210908566-e80acb65-c940-47d5-93d6-02de143883bb.png">

## Privilege Escalation
Once I have switched accounts, now I can go ahead and capture user flag.  
Now I will use comand **sudo -l** to check what the current user can run as root.  

<img width="538" alt="image" src="https://user-images.githubusercontent.com/114961392/210908764-7e9d6f20-7fb0-4bd0-9931-3759e01d7fb5.png">

This reveals that that user gwendoline can edit the file /home/gwendoline/user.txt with sudo privileges using /usr/bin/vi but the problem is that I can not run sudo with root (ALL , !root). if we had (ALL , ALL) then it would be possible.  
After some research, I found there is a vulnerability in sudo where if you run a user with -1, then sudo gets confused and reverts back to 0, which means root.  
**sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt**  

Once I enter this vim will load then I need to enter **!/bin/bash** and it will give me a root shell. From there I can go ahead and capture root flag.  

<img width="418" alt="image" src="https://user-images.githubusercontent.com/114961392/210909728-3cf4f626-f238-445a-9737-822da8241bf9.png">
