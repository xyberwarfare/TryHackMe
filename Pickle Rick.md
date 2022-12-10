# TryHackMe - Pickle Rick Room

<img width="539" alt="image" src="https://user-images.githubusercontent.com/114961392/206825186-ede3c63b-ff6c-44d8-825e-d37f5ad36996.png">

This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle.  

### What is the first ingredient Rick needs?

First, let's scan all ports with nmap.  

<img width="329" alt="image" src="https://user-images.githubusercontent.com/114961392/206825750-50672ebf-9861-40fb-8fdd-2b321eab115e.png">

Ports 22 (SSH) and 80 (HTTP) are open. Let's check out port 80 in the web browser.  

<img width="609" alt="image" src="https://user-images.githubusercontent.com/114961392/206825935-977eab6d-b1f0-459b-8cf0-c7e4d6300d62.png">

There is nothing too interesting on the web page other than a message telling us we need to find 3 incredients for Rick's potion. Let's check out the page source code.  

<img width="877" alt="image" src="https://user-images.githubusercontent.com/114961392/206826845-80139266-2c9f-4a38-8f9d-16c43a4f2129.png">

This has some interesting information down the bottom which gives us a username.  
Now let's use **dirsearch** to brute force any directories.  

<img width="474" alt="image" src="https://user-images.githubusercontent.com/114961392/206827250-26c9fa6f-8d56-4f08-bf93-8754e38c917b.png">

This shows 2 interesting directories:  

#### robots.txt
In this file is what looks like a password: **Wubbalubbadubdub**  

<img width="221" alt="image" src="https://user-images.githubusercontent.com/114961392/206827579-314969c9-66e2-473a-ac7c-f304a3b45db9.png">

#### login.php
This directory is a login page which the title **Portal Login Page**. Let's try using our discovered credentials here.  
This logs into a new page with an application that looks like can run commands.  

<img width="639" alt="image" src="https://user-images.githubusercontent.com/114961392/206827745-604a87c0-9a28-457c-a7f6-8caa20817a17.png">

Let's try the **ls** command. This works and shows us a list of the directory.  

<img width="582" alt="image" src="https://user-images.githubusercontent.com/114961392/206827796-160a0c26-df96-4c2b-a26a-e3409c78709d.png">

However, when I try using the **cat** command, we are presented with an error.  

<img width="581" alt="image" src="https://user-images.githubusercontent.com/114961392/206827892-dafb9cb1-37e8-4f46-816a-04c803940e5b.png">

Let's try to use bash to connect back to my host machine.  

<img width="581" alt="image" src="https://user-images.githubusercontent.com/114961392/206828262-60a9990d-78e7-4633-b064-8a0a5ab3d9b1.png">

<img width="402" alt="image" src="https://user-images.githubusercontent.com/114961392/206828274-de4c5e98-9a17-4288-8278-ee31f9381155.png">

This works and now we have a reverse shell. Now we can access the Sup3rS3cretPickl3Ingred.txt file which tells us the ingredient is mr. meeseek hair.  

### Whats the second ingredient Rick needs?

Before continuing, I will upgrade to a more stable terminal.  

<img width="313" alt="image" src="https://user-images.githubusercontent.com/114961392/206828446-5c77e206-6316-41a3-9351-a8f389a95c81.png">

Then let's traverse over to rick's directory in home and look at the second ingredient.  

<img width="319" alt="image" src="https://user-images.githubusercontent.com/114961392/206828533-286bf36a-adc7-47f8-89ed-f4adc0911558.png">

### Whats the final ingredient Rick needs?

Let's use the command **sudo -l** to check what we can use with root privileges.  

<img width="469" alt="image" src="https://user-images.githubusercontent.com/114961392/206828852-032062c8-6529-4c5e-a7f5-5a3ebffbdadb.png">

Looks like we can use everything with sudo without a password! Easy, now we can just use **sudo bash** to get a root shell.  
Now we can get the 3rd ingredient within the root directory.  

<img width="227" alt="image" src="https://user-images.githubusercontent.com/114961392/206828892-359c5a76-5a9f-4699-b0d4-1a0cece890a8.png">


