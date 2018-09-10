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

`ssh -p 2220 bandit8@bandit.labs.overthewire.org` with the password found.

Let's find a readable string beginning with several "=" characters in the file :

`strings data.txt | grep =====`.

The password is the longest string to print after the "=" characters.

