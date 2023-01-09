# TryHackMe - Smag Grotto Room
<img width="566" alt="image" src="https://user-images.githubusercontent.com/114961392/211225897-5e4eb8bb-7d2c-4798-97d6-54cc9e62ed67.png">

## Reconnaissance
Let's start by scanning all ports with nmap then run a version and script scan on all open ports.  

<img width="484" alt="image" src="https://user-images.githubusercontent.com/114961392/211225975-ce4263b8-0c0d-4639-ab82-80d6328ae9de.png">

This reveals that ports 22 (SSH) and 80 (HTTP) are open.  
Let's have a look at the webpage on port 80.  

<img width="425" alt="image" src="https://user-images.githubusercontent.com/114961392/211226004-14373c17-a63f-49cd-b3f5-0e55af3b8f18.png">

There is nothing interesting here or in the source code. Now I will try brute forcing for any hidden directories.  

<img width="473" alt="image" src="https://user-images.githubusercontent.com/114961392/211226113-43b23431-4eca-464f-b76c-10789ebfd907.png">

This reveals 2 interesting directories. The **/index.php/login** directory just redirects back to the home page however the **/mail** directory looks useful.  

<img width="803" alt="image" src="https://user-images.githubusercontent.com/114961392/211226184-5070f0d0-3423-4f58-a3ed-8051ccbd4995.png">

Here we can find a pcap file, 3 usernames (netadmin, jake, and uzi), and a virtual host domain (@smag.thm). I will note the usernames down, register the host name in my /etc/hosts file, and download the pcap file. At the top of the page it says we must use wget to download the pcap file so I will copy the address and grab through wget.  

<img width="807" alt="image" src="https://user-images.githubusercontent.com/114961392/211226293-8a46cfd1-14f4-4bb5-a116-1287c6c9b38a.png">

Once the pcap file has been downloaded, I can inspect it using wireshark.  
In wireshark, 4 packets down we can see that there are some credentials in the packet. I will right click on this packet and copy as printable text.  

<img width="929" alt="image" src="https://user-images.githubusercontent.com/114961392/211226411-f9cda573-6a2b-4478-815f-a28a5ed02e71.png">

Then I will paste this into the note pad so we can clearly read it.  

<img width="209" alt="image" src="https://user-images.githubusercontent.com/114961392/211226432-58ffd01b-9b87-4b4b-82e8-5ed614592fc9.png">

This reveals the username and password plus a new subdomain. I will double check this subdomain by fuzzing using FFUF.  

<img width="688" alt="image" src="https://user-images.githubusercontent.com/114961392/211226680-20d938f9-778c-4e00-857d-7ec57c95396d.png">

FFUF confirms the **development** subdomain so I will add this to my /etc/hosts file.  
Now let's check out the new domain.  

<img width="283" alt="image" src="https://user-images.githubusercontent.com/114961392/211226727-a442b4de-782e-4a00-9e77-b054c4aa2985.png">

Both the **admin.php** file and **login.php** go to the same login page. Here I will enter the credentials I found in wireshark.  
This logs in and now I am presented with a page where you can enter commands. Let's test this out.  

<img width="638" alt="image" src="https://user-images.githubusercontent.com/114961392/211226850-b324bf6b-5215-4688-b740-ca14e5baf9e7.png">

## Exploitation
I entered the **id** and **ls** commands however nothing happened. It looks like it is a blind command injection vulnerability where it does not show the output.  
To test if I can get a connection back to my machine, I will run **tcpdump** on my machine and run the ping command on the web page.  

<img width="637" alt="image" src="https://user-images.githubusercontent.com/114961392/211227030-89e35379-cab7-43ad-b234-1ab20c9a7380.png">

<img width="443" alt="image" src="https://user-images.githubusercontent.com/114961392/211227050-42e80462-debc-48ef-956e-4307a8c84e9c.png">

**tcpdump** displays the request and response from the ping command. Now that I know I can establish a connection to my machine, I will try a reverse shell.  
First, I will open a listener on my machine port 4444. Then I will use a basic bash reverse shell from [revshells.com](https://www.revshells.com/) on the web page.  

<img width="637" alt="image" src="https://user-images.githubusercontent.com/114961392/211227201-c237314b-66df-4b7e-bb90-196e4fa9f414.png">

<img width="399" alt="image" src="https://user-images.githubusercontent.com/114961392/211227209-a58a0613-f33e-49ce-9504-8cecd6d373d3.png">

This works and now I have a reverse shell!  
Before continuing, I will upgrade the terminal using the **stty** technique.  
**script /dev/null -c bash**  
**stty raw -echo; fg**  

<img width="361" alt="image" src="https://user-images.githubusercontent.com/114961392/211227294-3767dfd2-ac14-4248-b66d-78e5ff779a00.png">

Now that I am in, I will traverse to the /home directory where there is only 1 user, jake. In his folder is the user.txt flag however we do not have permission to read it. I can not access the .ssh folder either. Now I must escalate privileges.  

## Privilege Escalation
**sudo -l** requires a password and the history command has nothing of interest. However, the /etc/crontab file shows something useful.  

<img width="541" alt="image" src="https://user-images.githubusercontent.com/114961392/211227709-04897ec8-5abf-43d8-a9e7-f6f5da3fb33a.png">

This shows that every minute, root copies the contents of /opt/.backups/jake_id_rsa.pub.backup and writes it in /home/jake/.ssh/authorized_keys.  
I should be able to exploit this by using **ssh-keygen** to create a new pair key, then copying the public key to the backup file which root will then copy to authorized_keys.  

First, I will use **ssh-keygen** to make a new key pair.  

<img width="320" alt="image" src="https://user-images.githubusercontent.com/114961392/211227972-17e7f317-2520-49d8-877c-70116fcd8be2.png">

Then I will copy the contents of **jake.pub** (public key) to the backup file **/opt/.backups/jake_id_rsa.pub.backup** on the target machine using **echo**.  
After waiting 1 minute, the cron job should write the new public key to the authorized_keys file and I should be able to connect through SSH using the private key I created with ssh-keygen.  

<img width="417" alt="image" src="https://user-images.githubusercontent.com/114961392/211228257-e429fc70-cd07-4478-a0db-2ebae9bf606e.png">

This works and now we are connected through SSH with jake's account.  
Now I can go ahead and capture user flag.  

<img width="170" alt="image" src="https://user-images.githubusercontent.com/114961392/211228314-1a98ba16-d815-4fc4-8f8e-a1c307717350.png">

From here, I will use the **sudo -l** command to see what the user can run as root.  

<img width="599" alt="image" src="https://user-images.githubusercontent.com/114961392/211228490-fd45ddd6-8aa0-42d1-b4b7-f046ce72e542.png">

This shows that the **apt-get** binary can be run as root. I will head over to [GTFOBins](https://gtfobins.github.io/gtfobins/apt-get/) to see how to exploit this.  

<img width="644" alt="image" src="https://user-images.githubusercontent.com/114961392/211228372-4fcaf88e-4dc0-450b-a644-3d02b4d4283e.png">

I will try running the last command on GTFOBins page since we already have a shell, and WOOSHKAH! It works and now I have root privileges. From here I can go ahead and capture root flag.  

<img width="358" alt="image" src="https://user-images.githubusercontent.com/114961392/211228452-4dd29061-8478-4ad2-a2ec-f51efef46415.png">
