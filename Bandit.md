# Level 0

`ssh -p 2220 bandit0@bandit.labs.overthewire.org` with password `bandit0`

# Level 0 => 1

`cat` is a command line utility to show the content of a file. We want to show the content of the file `readme` in the current directory.

Command is `cat readme`

# Level 1 => 2

`ssh -p 2220 bandit1@bandit.labs.overthewire.org` with the password found.

Command is `cat ./-`

# Level 2 => 3

`ssh -p 2220 bandit2@bandit.labs.overthewire.org` with the password found.

Easiest way to complet this level is to use bash tab completion.

Type : `cat s<TAB>` then enter. Bash will auto complete the file name and escape the spaces.

# Level 3 => 4

`ssh -p 2220 bandit3@bandit.labs.overthewire.org` with the password found.

In Linux, a hidden file starts with a dot (e.g. `.secret_file`). It will not show in a file explorer or when using `ls` with no extra options.

There is two basic ways to show a hidden file :
- Using the `-a` flag of `ls` will show hidden file
- Using bash tab completion : `ls inhere/.<TAB><TAB>` will show all files and directory starting with a dot in the `inhere` folder

Let's see what is the hidden file of this level.
`ls -a inhere/` will show `.  ..  .hidden`
The file is named `.hidden`, to get the content : `cat inhere/.hidden`.

# Level 4 => 5

`ssh -p 2220 bandit4@bandit.labs.overthewire.org` with the password found.

let's go into the `inhere` directory and see what files are human readable :

`cd inhere/ && file ./*`

The `file` program will perform some read magic on all files of the directory and tell us what data type is in there.

We can see the only file containing ASCII text is `./-file07` so let's print it's content.

`cat ./-file07`

# Level 5 => 6

`ssh -p 2220 bandit5@bandit.labs.overthewire.org` with the password found.

We want to find a file with 1033 bytes in size and not executable by the current user.

We will issue the following command to filter those files and then check them manually for human readability.

`cd inhere/ && find -type f -size 1033c ! -executable`

There is only one file ! So the password in this file, `cat` it and get the password.

# Level 6 => 7

`ssh -p 2220 bandit6@bandit.labs.overthewire.org` with the password found.

We will again use `file` and filter with group, user and size filters :

`find / -group bandit6 -user bandit7 -size 33c`

Search for the only file with no "Permission denied" message and `cat` it.

# Level 7 => 8

`ssh -p 2220 bandit7@bandit.labs.overthewire.org` with the password found.

We search for the word "millionth" in the `data.txt` file :
`grep millionth data.txt`.

The password is next to the word.

# Level 8 => 9

`ssh -p 2220 bandit8@bandit.labs.overthewire.org` with the password found.

We will sort the file by dictionnary order and then extract the only unique line :

`sort -d data.txt | uniq -u`

The password is the only line to print.

# Level 9 => 10

`ssh -p 2220 bandit9@bandit.labs.overthewire.org` with the password found.

Let's find a readable string beginning with several "=" characters in the file :

`strings data.txt | grep =====`.

The password is the longest string to print after the "=" characters.

# Level 10 => 11

`ssh -p 2220 bandit10@bandit.labs.overthewire.org` with the password found.

The `base64` utility can encode and decode data. The `-d` flag is for decoding do we will pipe the output of the file to this :

`cat data.txt | base64 -d`

# Level 11 => 12

`ssh -p 2220 bandit11@bandit.labs.overthewire.org` with the password found.

Using `tr` to translate ROT13 : `cat data.txt | tr '[A-Za-z]' '[N-ZA-Mn-za-m]'`

# Level 12 => 13

`ssh -p 2220 bandit12@bandit.labs.overthewire.org` with the password found.

Let's start by creating and going into a temporary directory : `cd $(mktemp -d)`

Copy the file to the current directory : `cp ~/data.txt .`
Reverse the hex dump : `xxd -r data.txt > raw.bin`
And then check what type of file we have : `file raw.bin`

This is a GZ archive. Let's rename it and decompress it : `mv raw.bin raw.gz && gzip -d raw.gz`.

Let's check what type of file we now have : `file raw`.
This is a BZIP2 archive, let's decompress it : `bzip2 -d raw`

Let's check what type of file we now have : `file raw.out`.
Let's rename it and decompress it : `mv raw.out raw.gz && gzip -d raw.gz`.

Let's check what type of file we now have : `file raw`.
This is a TAR archive, finally something new !

Continue this until you have a file which output `ASCII text` with the file command.

# Level 13 => 14

Let's get the private key we need to log into level 14 : 

`scp -P 2220 bandit13@bandit.labs.overthewire.org:/home/bandit13/sshkey.private . && chmod 600 sshkey.private`

And then use it : `ssh -p 2220 -i sshkey.private  bandit14@bandit.labs.overthewire.org`

# Level 14 => Level 15

Send the current level password to localhost:30000 : `cat /etc/bandit_pass/bandit14 | nc localhost 30000`

# Level 15 => Level 16

`ssh -p 2220 bandit15@bandit.labs.overthewire.org` with the password found.

`cat /etc/bandit_pass/bandit15 | openssl s_client -ign_eof -connect localhost:30001`

# Level 16 => Level 17

`ssh -p 2220 bandit16@bandit.labs.overthewire.org` with the password found.

Let's find the open ports : `nmap localhost -p 31000-32000`.

There are 5 open ports : 

```PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown
```
We need to find the only one to send a different text, we will issue those commands :

- `cat /etc/bandit_pass/bandit16 | nc localhost 31046`
- `cat /etc/bandit_pass/bandit16 | nc localhost 31518`
- `cat /etc/bandit_pass/bandit16 | nc localhost 31691`
- `cat /etc/bandit_pass/bandit16 | nc localhost 31790`
- `cat /etc/bandit_pass/bandit16 | nc localhost 31960`

We notice that ports 31518 and 31790 don't send anything, it means that they speak SSL. The other ports all sends the same text so they are not relevant.

- `cat /etc/bandit_pass/bandit16 | openssl s_client -ign_eof -connect localhost:31518` => Still not correct

- `- `cat /etc/bandit_pass/bandit16 | openssl s_client -ign_eof -connect localhost:31518` => We get an RSA private key !

Copy and past the key in the `sshkey.private` that we created earlier.

# Level 17 => 18

`ssh -p 2220 -i sshkey.private   bandit17@bandit.labs.overthewire.org`

The diff utility show the differences between two files :

`diff passwords.old passwords.new`, the password is on the last line of the output.

# Level 18 => 19

`ssh -p 2220 bandit18@bandit.labs.overthewire.org` with the password found.

We can append a command after the SSH line to execute it. Let's do it :

`ssh -p 2220 bandit18@bandit.labs.overthewire.org cat readme` and we get the next password !

# Level 19 => 20

`ssh -p 2220 bandit19@bandit.labs.overthewire.org` with the password found.


We have an executable named `bandit20-do` in our home directory.
What does it do ?

```bandit19@bandit:~$ ./bandit20-do 
Run a command as another user.
  Example: ./bandit20-do id
  ```
So it runs a command as bandit20. We can get the next passord like this :
`./bandit20-do cat /etc/bandit_pass/bandit20`

# Level 20 => 21

`ssh -p 2220 bandit20@bandit.labs.overthewire.org` with the password found.

First, create a listening server on port 50000 (arbitrary number) :

`nc -l 50000`.
Hit <CTRL-Z> to pass the task in background.
Then create the client : `./suconnect 50000`.
Hit <CTRL-Z> to pass the task in background.

`fg 1` to go back to netcat, paste bandit20 password into it and hit enter to send it.

`fg 2` to return to suconnect, it will show that the next password was sent back.

`fg 1` to return again to netcat, the next password will be there.

# Level 21 => 22

`ssh -p 2220 bandit21@bandit.labs.overthewire.org` with the password found.

Let's look into `/etc/cron.d/` : `ls /etc/cron.d/`
Looks like there is an interesting file, what does it contains ?
`cat /etc/cron.d/cronjob_bandit22`
It shows a script being executed at each reboot and every minutes :
`/usr/bin/cronjob_bandit22.sh`

What does it do ?
`cat /usr/bin/cronjob_bandit22.sh` => It copy the bandit22 password into the file `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv`.

Finally, `cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv` will give us the next password.
Yk7owGAcWjwMVRwrTesJEwB7WVOiILLI
