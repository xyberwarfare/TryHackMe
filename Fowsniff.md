# TryHackMe - Fowsniff Room
<img width="569" alt="image" src="https://user-images.githubusercontent.com/114961392/210707106-1ee2711e-69fe-4406-95bc-bb83f68d077d.png">

## Reconnaissance
First of all I will scan all ports with nmap then run a version and script scan on open ports.  

<img width="763" alt="image" src="https://user-images.githubusercontent.com/114961392/210704153-d690987e-8329-44a7-a9f8-8190a2600993.png">

Ports 22 (SSH), 80 (HTTP), 110 (POP3), and 143 (IMAP) and open.  
Let's have a look at the webpage.  

<img width="627" alt="image" src="https://user-images.githubusercontent.com/114961392/210704706-dd92c271-2b09-4962-a456-f4a3e2a406d0.png">

There is nothing too interesting here or in the source code except a reference to their twitter page:  
**The attackers were also able to hijack our official @fowsniffcorp Twitter account. All of our official tweets have been deleted and the attackers may release sensitive information via this medium. We are working to resolve this at soon as possible.**  
Let's investigate their twitter.  

<img width="327" alt="image" src="https://user-images.githubusercontent.com/114961392/210706182-1d427b52-d704-4c22-a2c4-a4a4ebdec441.png">

Looks like the attackers dumped all their passwords in a pastebin link. Let's download and have a look.  

<img width="433" alt="image" src="https://user-images.githubusercontent.com/114961392/210706542-e232d20a-c98f-4828-847b-4db74655d412.png">

I will copy the usernames to a file (users.txt) and see if I can crack the hashed passwords with either crackstation.net, hashcat, or john the ripper.  

<img width="782" alt="image" src="https://user-images.githubusercontent.com/114961392/210707915-d7463852-516e-4136-8d6b-b06951ab6940.png">

Crackstation cracked all except 1 which is fine, I should be able to work with this.  
Next let's try brute-forcing POP3 with these password and usernames using Metasploit as the TryHackMe hint suggests:  
**In metasploit there is a packages called: auxiliary/scanner/pop3/pop3_login where you can enter all the usernames and passwords you found to brute force this machines pop3 service.**  

Let's fire up Metasploit and start brute-forcing!  
First I will make sure the usernames and passwords are in 2 seperate files. Make sure to remove the @fowsniff after each username otherwise it will fail. Then search pop3 in Metasploit and use the pop3_login module. After setting the options I will run this and let it do it's thing.  

<img width="833" alt="image" src="https://user-images.githubusercontent.com/114961392/210709851-b562bb49-2809-4756-87cb-d7afccec649d.png">

<img width="570" alt="image" src="https://user-images.githubusercontent.com/114961392/210710693-94036cdf-1867-4fe9-bcb9-d0fc3f7b0558.png">

Sure enough this reveals siena's password.  
**What was seina's password to the email service?**  
scoobydoo2  

Now we can try log into POP3 using these credentials. I will use netcat to connect and do a little researching on how to use the POP3 commands.  

<img width="577" alt="image" src="https://user-images.githubusercontent.com/114961392/210710908-f62f8fdf-f83c-403f-a00c-c6a5fdf4cb58.png">

The first email reveals the temporary password is S1ck3nBluff+secureshell.  

<img width="431" alt="image" src="https://user-images.githubusercontent.com/114961392/210711159-b172c629-fbc5-46c1-b8e9-49e9783b1bca.png">

-------
