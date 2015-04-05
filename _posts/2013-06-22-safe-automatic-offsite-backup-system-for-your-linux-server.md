---
layout: post
title: Safe, automatic offsite backup system for your linux server
---

How to backup a server

-----

Linux is the most common operating system used when we talk about servers. It's all over the internet. Linux platforms are free, robust and very stable. You, as a server owner, are definitely worried about your server's security and stability. Any sysadmin must worry about backups and make sure that the data on the server is safe even if the hard drive suddenly dies. This is one of those things that distinguishes a good sysadmin from a poor one. Believe me: even though I fortunately never had to do it, you don't want to have to say to your boss / customer that all their data is gone, simply because YOU didn't do your job properly. Not taking care of backups and trusting that nothing will go wrong is the most stupid mistake you can make, and you will regret it every single minute of your life when you realize that all your data is lost just like that, in less than one second.

Oh, and let me tell you something, before I forget:

**RAID is not a backup mechanism!!!**

For some reason, lots of people think that their server is secure because they have RAID. RAID is another world, it just saves your server from being down when your hard drive fails and you must replace it with a new one. I'm not going to cover RAID here, for the sake of simplicity, it's enough for you to know that there are lots of RAID setups; the most common objective is to replicate or distribute data in more than 1 disk, so that you're in safe hands when one of them fails.

RAID is not a backup mechanism because it doesn't account for accidental data removal. Remember, RAID replicates your data. What if someone erases the entire home directory? Well, RAID will just propagate this to every disk. What if a bug in some program corrupts a file under /etc, or what if you want to go back to the old config you had a week earlier? There can be lots and lots of situations where RAID doesn't help you, because it is not supposed to work as a backup system. A good backup system must follow some rules. Over the course of time, I have defined some backup rules that seem to me pretty satisfactory. For me, a good backup system is something that:


* is kept in a private, separate disk or partition not used for anything else
* only your user account, or the backup user account, has access to that disk or partition
* is updated to an offsite safe location frequently, far from your server
* keeps numerous versions of your data, allowing you to go back at least 2 weeks
* makes a smart use of available space by making incremental versions of files instead of copying them everytime a backup is made
* does not slow down or disrupt normal server operation

Keeping these points in mind, today I want to show you how I backup my own dedicated server.

From a high level point of view, my backup system is composed of a few number of components:

* A daily cron job that runs a script to make backups to the backup partition
* A cron job, executed at night, that transfers the backups of the day to an offsite location
* The offsite location, which receives the backups
* A cron job in the offsite location to make backups of the backup to an external drive

As you can imagine, you can do it as recursive as you want (like backups of the backups to another offsite location, which in turn makes backups to another place, etc., depends on how paranoid you are).

At this point, you may be wondering what is my "offsite location". Well, my offsite location is just my home server.

Ok, now that I've mentioned the components, we need to talk about the tools. For my backup system, I use 2 useful tools: `backup-manager` and `rsync`. `backup-manager` is a little perl script that automates your local backups and that can easily be added to a cron job. Virtually every linux distribution provides `backup-manager` as an easy-to-install package. All you have to do is edit the configuration file to your needs and run it periodically with crontab. I'm no going to cover crontab or `backup-manager` configuration in this article, I assume that is an easy task (just make sure you configure `backup-manager` to use tarball-incremental backups, or you will end up having a lot of space taken up if you want to keep backups for the last 2 weeks). There are lots of good pages on the internet covering both topics.

I do, however, want to emphasize an important point that I have learned by myself over the course of time, and by asking other experienced sysadmins, which is the question of which folders you should backup. On a normal linux server, I recommend backing up the following folders:

* `/etc` - This will make sure you will not lose your configuration files.
* `/home` - Obviously, every user files should be backed up
* `/var` - This is an important folder because user settings (like crontab entries) are kept here, as well as system logs. In case of a disaster where you suddenly lose your server, you will want to have copies of all your server logs prior to the disaster so that you can analyze them really carefully
* `/boot` - contains kernel image and configuration, it's important if you need to restore your system
* `/root` - root's home is like any other user's home, make sure you include this directory

Depending on the level of catastrophe you're trying to avoid, you may also want to include a backup of `/usr`, where a big part of software is installed.

These folders should be indicated in the configuration of `backup-manager`. The idea is to run `backup-manager` once (or more) every day using a tarball-incremental configuration to generate tarballs of all of the folders you chose to backup and place them in a separate partition or disk where no one else can touch. 

And now, the best part comes: uploading it to an offsite location. Lots of things can go wrong with your server or datacenter. You never know. My point is that you can't trust that everything you put in your server is kept there forever, and that there will be no problems at the datacenter, or in the server's disk, or whatever. Just assume that your datacenter is not a reliable place to keep files in a persistent way. You'd better copy your backups and take them out of there asap. It's a mistake to keep all of the backups in your remote server. Just bring them home. It's not that hard.

To bring my backups home, I use `rsync`. As you can imagine, my backup system doesn't require any kind of human intervention, but this brings up a security issue, because `rsync` must be able to authenticate to the other end automatically. Sure, we have public key authentication between 2 linux systems, but I don't want to open that can of worms to the rest of the world, if you know what I mean. I tend not to trust very much on having passwordless authentication on my server. It would be really bad if someone was able to grab one of my keys and have immediate access to one of the shells at my server. To fix this, I created a security policy consisting of 3 rules:

* Passwordless authentication will be made only on my home server. This means that my remote server at the datacenter will connect to my home server, and not the contrary. I made it this way because my home server is most likely not very interesting to hack into, and doing so would really not have terrible effects. It would just be a pain in the ass.
* Passwordless authentication only allows execution of a single specific `rsync` command that syncs remote (from datacenter) backups to my home server. This way, even if someone gains access to the key and authenticates to my home server, all they can do is simply run the rsync utility to pick up the new files from the remote server. Not very useful.
* Passwordless authentication is only allowed if the request is coming from my remote's server ip.

Of course that this is not 100% safe (not even close), but it surely makes an attacker's life much, much harder.

This is not a very complex set of rules to enforce; as you will see, it can be quickly made with a couple of commands. I will show you how to do this, and I will also teach you a clever trick that you can use for yourself to force the execution of any command you choose, not just `rsync`.

From now on, I will call my home server (offsite location) `raspberrypi`, and my dedicated server `server1`. Let's imagine that we're using an account named `foo` to manage backups in both machines.

So let's start with the first one. The first step is to setup passwordless authentication. To do this, we need to generate a pair of keys in `server1`. You can do this using `ssh-keygen` command. Let's store the keys inside our private `.ssh` directory (if it doesn't exist, create it and make sure permissions are set to `700`).

```
foo@server1:~$ cd .ssh
foo@server1:~/.ssh$ ssh-keygen -f backups
```

Do not enter a passphrase (that's what makes authentication automatic, otherwise you would have to enter the passphrase everytime you wanted to use this key to authenticate). This command will create 2 files, `backups` and `backups.pub`. The pub file is the public key, the other is the private key. The private key is secret and should never leave `server1`. The next step is to make the public key known in `raspberrypi`. To do this, we will use `ssh-copy-id` command:

```
foo@server1:~/.ssh$ ssh-copy-id -i backups.pub foo@raspberrypi
```

If `raspberrypi` listens on a non-default SSH port, you can use this instead:

```
foo@server1:~/.ssh$ ssh-copy-id -i backups.pub "foo@raspberrypi -p port-number"
```

This command should ask you for foo's password at `raspberrypi`. When you enter the password, it will append the public key to foo's authorized_keys file at `raspberrypi`. On success, it will tell you this:

```
Now try logging into the machine, with ssh 'foo@raspberrypi', and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.
```

You are free to do so, but `ssh-copy-id` never copied any extra information I wasn't expecting, so don't pay much attention to that "warning". Still, it's worth the check.

By now, you should be able to log into `raspberrypi` from `server1` without password authentication. You just have to specify the appropriate private key file. To make sure everything is working so far, try a remote login session from `server1`:

```
foo@server1:~$ ssh foo@raspberrypi -i backups
```

This should give you a shell prompt on `raspberrypi` without asking for a password. Step 1 is completed.

The next step is to allow only execution of the `rsync` command when someone uses this key to login without a password. Better yet, we will not allow generic `rsync` usage, we will basically force execution of the `rsync` command that uploads backup files to `raspberrypi`. That's even more precise, and that's all we want to do. First, write down the `rsync` command you will want to use on `server1`. For example, I store local backups in `server1` under `/mnt/backups/server1`, and I keep that directory synced with `/home/foo/srvbackups` in `raspberrypi`. To do so, I use this `rsync` command:

```
foo@server1:~$ rsync -az --delete --rsh='ssh -p port-number -i /home/foo/.ssh/backups' /mnt/backups/server1 foo@raspberrypi:~/srvbackups/
```

You can remove `-p port-number` if `raspberrypi` listens on the default ssh port. When executed in `server1`, this will mirror the local backups folder on `server1` to the remote folder `srvbackups` on `raspberrypi`. The `--delete` option is used to force full mirroring; without `--delete`, files deleted on `backups` folder inside `server1` are not deleted in the remote folder from `raspberrypi`. You can choose not to use `--delete`, but make sure you have some other way to purge really old files from `raspberrypi`, or you can run out of space. `-az` are options to compress the files and make the file transfers faster, more details are provided in the manpage.

Now, how can we force execution of this command in `raspberrypi` side? First of all, how do we force execution of a command when someone uses the `backups` key to login?

All you have to do it edit the `authorized_keys` file. It should be under `.ssh/` directory on foo's home (it may be called `authorized_keys2` as well). This file accepts a set of options before the key declaration. Just open up `authorized_keys` from foo on `raspberrypi` in your favorite editor, and search for the last key in the file (which corresponds to the one we just added; `ssh-copy-id` appends keys to the end of the file). You will see something like this:

```
ssh-rsa AAA...<lots of characters> foo@server1
```

We can add options for this key's usage in the beginning of the line, before `ssh-rsa`. To force a command, you must do something like this:

```
command="<command-here>" ssh-rsa AAA...<lots of characters> foo@server1
```

Let's try this out. Edit the file to force execution of `ls` command. It will stay like this:

```
command="ls" ssh-rsa AAA...<lots of characters> foo@server1
```

Now, go back to `server1` and try logging into `raspberrypi` using our backups key file:

```
foo@server1:~$ cd .ssh
foo@server1:~/.ssh$ ssh -i backups foo@raspberrypi
srvbackups  word  word.c  word.c.save
Connection to raspberrypi closed.
foo@server1:~/.ssh$
```

Woow! You can see that foo's home directory on `raspberrypi` has got `srvbackups`, and a small C program named `word.c`. Note that we weren't able to actually do anything in `raspberrypi`: we got in there, `ls` was executed, and then you got kicked out.

What if we try to execute something else? Well, we don't have the chance to get a terminal on `raspberrypi`, but we can try to pass a command to ssh. For example, let's try to remove `word.c`:

```
foo@server1:~/.ssh$ ssh -i backups foo@raspberrypi "rm word.c"
srvbackups  word  word.c  word.c.save
Connection to raspberrypi closed.
foo@server1:~/.ssh$
```

Yeah, `raspberrypi` just ignores whatever you try to do. Right? Does it?

NO, IT DOESN'T!!!!!!

Actually, `raspberrypi` KNOWS what YOU tried to execute, but it doesn't execute it because there's a forced command inside the key file. And the best part is that in `raspberrypi` you can easily know what was the original ssh command that the other end tried to execute. This is perfect for logging purposes, but also for our goal, which is to force `rsync` usage for copying the backup files. Just to be clear, our problem here is that we don't know which command should we force. Sure, you know your `rsync` command on `server1` side, but what is transmitted to `raspberrypi` is not exactly the same command you issued on `server1`. For example, `rsync` on `raspberrypi` side doesn't really care that you're copying from `/mnt/backups/server1`, so that information is not passed along to the remote session inside `raspberrypi`.

Let's exploit this original ssh command thing to know what is passed to `raspberrypi` when you issue that long `rsync` command in `server1`. To do so, let's make use of a very simple, and also very useful, bash script:

```bash
#!/bin/sh

echo "Original SSH command is '$SSH_ORIGINAL_COMMAND'";
exit 0;
```

Save this as `ssh-original-command` inside `.ssh` folder. Make sure you make it executable (`chmod +x ssh-original-command`), and force execution of this script to whoever logs in using our backups key. In other words, `authorized_keys` file should look like this in the line of backups key:

```
command="/home/foo/.ssh/ssh-original-command" ssh-rsa AAA...<lots of characters> foo@server1
```

You can now play with this. Let's attempt to remove `word.c` again:

```
foo@server1:~/.ssh$ ssh -i backups foo@raspberrypi "rm word.c"
Original SSH command is 'rm word.c'
foo@server1:~/.ssh$
```

Niiiiiiiiiiiice! Oh, a quick side note: I have found that sometimes, with more complex commands, it doesn't print them properly in the terminal. You might want to add output redirection to a temporary file in `ssh-original-command` script, like this:


```bash
#!/bin/sh

echo "Original SSH command is '$SSH_ORIGINAL_COMMAND'" > results.tmp;
exit 0;
```

But then you have to login with normal ssh session to `raspberrypi` and open `results.tmp` file to find out the original command.

This is very powerful. Using this technique, you can now find out how ANY command looks like when it reaches the remote session. In particular, using this method, you can see that our `rsync` command on `server1` is passed to `raspberrypi` like this:

```
rsync --server -vlogDtprze.iLsf --delete . ~/srvbackups
```

This is the the request that `raspberrypi` receives when you execute that long `rsync` command in `server1`. Note the cryptic, undocumented set of options. `--server` probably makes `rsync` run in server mode. And also note that the source directory (`/mnt/backups/server1`) is not passed along (as I said, the remote machine doesn't really care about it).

So now, we can go back to `authorized_keys` file and change it to force execution of this `rsync` server-mode command. Here's how it looks like in the end:

```
command="rsync --server -vlogDtprze.iLsf --delete . ~/srvbackups" ssh-rsa AAA<lots of chars...> foo@server1
```

What's the catch? Well, if you have very specific needs and you want to execute lots of different commands, you have to setup a different key for each command and do these steps for each command. As an alternative, you can also use linux's built-in feature to run multiple commands sequentially, either using `;` (like `ls; cd bar; cat file.out`), or using `&&` (like `ls && cd bar && cat file.out`). The `&&` option is a little different from `;` because it only executes the next command if the previous was successfull.

Hurrah! We're now ready to go to the final step: making sure that only `server1` can use this passwordless authentication mechanism! This will turn out to be really simple. It's just another option that we need to add inside our `authorized_keys` file from `raspberrypi`, known as the `from` option. `from` works like `command` (the one we used before to force commands), and you can list all of the ips or hosts that are allowed to login using this key. The list is separated by commas, and it allows you to use wildcards (`*` to match arbitrary number of characters, `?` to match a single character). You can also use from to disallow only certain hosts by prefixing them with `!`. For example, this:

```
from="1.2.3.4,server1,*.example.com,?mail.com"
```

Will allow incoming logins using this key from ip `1.2.3.4`, or `server1`, or one of `example.com` subdomains, or any domain that has a single letter followed by `mail.com`, like `gmail.com`. On the other hand, this:

```
from="!host1.co.uk"
```

Will allow anyone to connect except `host1.co.uk`. Options inside `authorized_keys` file are separated by commas as well, so our file will stay like this:

```
from="server1",command="rsync --server -vlogDtprze.iLsf --delete . ~/srvbackups" ssh-rsa <key with lots of chars...> foo@server1
```

Just to make this safer, you can also include more restrictive options like `no-port-forwarding`, `no-X11-forwarding`, `no-agent-forwarding` and `no-pty`. You can learn about it with a quick google search, so I will not cover them in here.

Thus, our final version for our key entry is:

```
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty,from="server1",command="rsync --server -vlogDtprze.iLsf --delete . ~/srvbackups" ssh-rsa <key with lots of chars...> foo@server1
```

A very interesting option that we're not going to use but that I'd like to tell you about is the environment option. The syntax is similar to the other options I showed you:

```
environment="VAR1=VALUE1,VAR2=VALUE2"
```

This lets you define environment variables. You can change `$HOME` or `$PATH`, or add new variables. It's up to you.

With this, our backup system is nearly complete. Now, all you have to do is adding that original long `rsync` command on `server1` crontab. Remember that crontab doesn't use the same environment variables that you use when you open a terminal session, so don't forget to include the whole path to rsync (most likely `/usr/bin/rsync`). After that, all you have to do is sit down and enjoy your automatic backup system. Remember to make a regular check just to make sure that everything is working smoothly. You can also set another cron job in `raspberrypi` to sync the backups directory from your dedicated server to an external hard drive - that will give you another redundancy level. And you can apply this whole procedure recursively - you could now backup `raspberrypi` to ANOTHER offsite location using the same method to get one more redundancy level. Again, it depends on how paranoid you are.

Oh, and one more thing: test your backup system. It is easy to think that everything is safe until some day you need to use your backups and you find out that you forgot some `rsync` option, or any other thing, and basically your backups are useless. Therefore, it is important that from time to time (say, once every month or two) you make a virtual restore of your files in a testing machine (it can be `raspberrypi`), and see if everything is working like it should.

If you are really pedantic about your data and want extremely high fidelity offsite backups, there are plenty of services out there. You just have to pay. Amazon S3 is very popular, just google around and you will find lots of alternatives to my cheap home-server based solution, which, even though I find it pretty comfortable and cheap (and it's enough for me), I recognize that it is not nearly as stable as a professional service like Amazon S3.

