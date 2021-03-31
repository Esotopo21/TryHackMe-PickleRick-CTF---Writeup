# TryHackMe-PickleRick-CTF---Writeup
Writeup for pickle rick ctf challenge by TryHackMe 

Go visit TryHackMe website for more ! 
https://www.tryhackme.com/


We need to find the three ingredients, meaning three flags.
I'm gonna start enumerating the machine:
`nmap -p 1-5000 -A -Pn <MACHINE-ip>`

I found two open ports:
> 22/tcp open  ssh     OpenSSH 7.2p2 

> 80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))

It's worth taking a shot with hydra for the user root

`hydra -l root -t 16 -P usr/share/wordlists/rockyou.txt <MACHINE-ip> ssh`

> [ERROR] target ssh://<MACHINE-ip>:22/ does not support password authentication (method reply 4).

Let's visit the web server.

There is nothing interesting on the page, let's inspect the souce code.
There is a comment which reveals us a username: R1ckRul3s.

It doesn't match the standard ssh naming, thus user password/authentication is not supported. There must be a login form around the site, let's use dirb to enumerate it.

`dirb http://<MACHINE-ip>`

We found these directories:
/assets/
/index.html
/robots.txt
/server-status (403)

Let's take a look to the /assets dir. 

There are the minified js and css for bootstrap and jquery along with other images not used in the homepage. Those images together with the username we found earlier indicate that there must be a login form somewhere. 

Let's take a look to the robots.txt as well, as it can contain useful information.

It contains only one word: *censored* . This could be either an easter egg or a password, let's note it and move on.


At this point I tried to enumarate the web server with gobuster looking for php extensions

`gobuster dir -x php -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u <MACHINE-ip>`

A /login.php immediately pops up. Let's visit it.

It is a simple login page, let's try with what we have found so far (R1ckRul3s:*censored*) 

It works!

Let's explore the directory we are in.

`ls`

Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt

It looks like the first file listed could be our first ingredient!

`cat` and `more` are going to fail but `less` will do it!

Let's look at the file "clue.txt" 

> Look around the file system for the other ingredient.

So I guess we're going to look around. I won't use "find" as it seems that the flags naming is not standard naming, so it will be a waste of time.

Let's see who are the users in the machine

`ls /home/`

>rick

>ubuntu

Let's list the content of rick's home

`ls /home/rick`

> second ingredients

let's less it and we'll have the second ingredient !

Now we need to find the third and last ingredient. Being the first one in the current directory and the second in a user's home (I also tried to list the content of ubuntu but nothing showed up), we can presume the third one is in /root.

Let's see what are our privileged command

> User www-data may run the following commands on ip-10-10-88-141.eu-west->1.compute.internal:
>    (ALL) NOPASSWD: ALL
    
It means that we can execute al command as super user with no need to prompt a password.

Let's list the content of the /root directory with sudo

`sudo ls /root`

we see that our last ingredient is here ! let's sudo less it !

It works and we're done ! :)




