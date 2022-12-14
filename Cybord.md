# TryHackMe - Root Me Room
<img width="550" alt="image" src="https://user-images.githubusercontent.com/114961392/207510558-15e5c072-2234-43f1-9be2-635f537f5d06.png">

## Reconnaissance
Scan all ports with nmap then use service and script scan on open ports.  

<img width="496" alt="image" src="https://user-images.githubusercontent.com/114961392/207510914-5c423109-0c49-471a-ae15-b08909f6851f.png">

The only ports that are open are 22 (SSH) and 80 (HTTP). Let's check out the web page.  

<img width="401" alt="image" src="https://user-images.githubusercontent.com/114961392/207511109-0abf9ac9-856e-42c2-8444-d065931b64e4.png">

It is just the Apache Ubuntu default page. There is nothing interesting in the source code so let's try brute forcing directories.  

This reveals 2 interesting directories, **/admin** and **/etc**.  
The admin page has a site for music achievements with an interesting **Admin Shoutout** section and a downloadable archive.  

<img width="716" alt="image" src="https://user-images.githubusercontent.com/114961392/207511988-16df37c7-4741-4488-ae03-4a7e28db071a.png">

In the Admin Shoutout section we learn 3 user names (Josh, Adam, and Alex) and that Alex's credentials could be lying around somewhere.  
In the /etc directory are 2 files. The **passwd** file is interesting as it looks like a hash. Let's try crack it with **john**. First I will put the hash into a file then I will crack with john. You could also use **hashcat** for same result.   

<img width="491" alt="image" src="https://user-images.githubusercontent.com/114961392/207513249-f4312fe4-9098-4f59-b2f8-d951f46edbb9.png">

This reveals the hash is **squidward**.  
Now let's check out the downloaded archive. I will extract the archive with tar **(tar -xf archive.tar)** and traverse through the extracted directories. In the README file it makes a reference to **Borg Backup**.  
After some research into how to run Borg Backup, I created a new directory called mount then mounted using Borg with our cracked password.  

<img width="397" alt="image" src="https://user-images.githubusercontent.com/114961392/207515052-d5d46a46-9b55-48fc-83f8-a69a3e1d23e0.png">

Once the archive is mounted, I traversed the directories and found **secret.txt** in /Desktop which says:

<img width="356" alt="image" src="https://user-images.githubusercontent.com/114961392/207515483-909a55e9-c7ef-4537-ac32-65d8649b238a.png">

Then in the /Documents directory I found **note.txt** which reveals a username and password!  

<img width="478" alt="image" src="https://user-images.githubusercontent.com/114961392/207515652-13d8567e-61d3-493c-a744-e804ed508f7b.png">

## Exploitation
Let's try to connect through SSH with these credentials.  

<img width="466" alt="image" src="https://user-images.githubusercontent.com/114961392/207516123-39b5dc27-a988-4197-a205-4259e22deef5.png">

It works and now we can capture the user flag.  

## Privilege Escalation
### Method 1

After we have gained access, let's use command **sudo -l** to see what the user can run as root.  

<img width="599" alt="image" src="https://user-images.githubusercontent.com/114961392/207518779-05492126-56c0-4cd0-a194-ac67f8ca83e8.png">

It looks like we can run /etc/mp3backups/backup.sh. Let's have a look at the file.  

<img width="833" alt="image" src="https://user-images.githubusercontent.com/114961392/207518970-ac50860d-ba90-4422-968e-cb7979f0a5ca.png">

It is a script to backup all the music files. However it looks like we can break out of the script using the -c flag and issue commands. When I check out the bash history some interesting commands come up which confirms my theory.    

<img width="598" alt="image" src="https://user-images.githubusercontent.com/114961392/207519400-96e65d66-3e19-43b4-97fa-9de99edee0a7.png">

I will try copying one of the history commands to see what happens.  

<img width="833" alt="image" src="https://user-images.githubusercontent.com/114961392/207519717-e5b52349-ee1e-4bc4-a722-bd488b8fe854.png">

Perfect, at the end you can see it executes the **whoami** script.  
Now I will replace command **whoami** with **/bin/bash cat /root/root.txt** and hopefully this should display the root flag. Once I have run the command it finishes and now the user has changed to root.  
I will try cat /root/root.txt again then exit and WOOSHKAH! We have the root flag.  

<img width="833" alt="image" src="https://user-images.githubusercontent.com/114961392/207520372-5c32fcce-f68c-4ca9-adeb-48f2768881ef.png">

### Method 2
If we have a look at the permissions for the backup.sh file, we can see it does not have write privileges. Since this is owned by alex, we can change this to run all privileges then change the script to **/bin/bash** to create a new shell.  

<img width="336" alt="image" src="https://user-images.githubusercontent.com/114961392/207521740-9d01b7ef-de0e-4d81-84ba-18ac32b57442.png">

Then we just need to run the script from sudo. Now we have root access and can capture root flag.

<img width="249" alt="image" src="https://user-images.githubusercontent.com/114961392/207521987-9b918765-5f79-40a6-9319-941bd3ad0dd7.png">
