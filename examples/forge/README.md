### This guide will allow you to run a Forge (modded) Minecraft Server on Akash 

***
## First Step

## Setup an "ssh" ubuntu image on [ Akash ](https://akash.network)  

## Optional : build custom Docker image (recommended)
Update Dockerfile with your needs.
```
# Note you can also use the prebuilt image at moonbys/moonubu:java8
$ docker build -t $your_dockerhub_user/ubuntu:1 .
```
Update deploy.yaml with your newly built docker image. ( or keep the prebuilt image)
## Buy me a coffee to keep the work going

| | |
--- | --- 
|akash:|`akash178d5wcg95z2jc5au608a9scshvwy3stwt3cgmq`|

### Buy Coffeeroaster a coffee to keep his work going and for https://github.com/coffeeroaster/akash-ubuntu

| | |
--- | --- 
|akash:|`akash1k07dv0wcjej4e4a5z9dg3c6yvvf9mcnz0sacp9`|
|monero:| `88z2xaJaRPVc5VXtE3CArxeAkqSQV8aA4EhswR8pqfY3CghQfaD7nYsLvmcnXuHj1TYDeJaGvyyyW9XyX6ka7BLzQ7ypmqJ`|

## Prepare your ssh session

On your workstation setup an ssh public/private key pair.
```
$ssh-keygen -t rsa
# (a password is optional. But in most cases you don't need one).
# Note that $HOME/.ssh/id_rsa.pub is now created
```

## Update deploy.yaml

Copy and paste the contents of `$HOME/.ssh/id_rsa.pub` and paste them into deploy.yaml (env -> pubkey). (deploy.yaml : line 8).

NOTE: Make sure that you do not use any `"` around the pubkey as this will cause problems.


### Update resources as needed. Set the amount of RAM / CPU / diskspace as needed.

## Deploy to Akash
* This assumes that you have installed akash in `$HOME/bin/akash`
* Read through https://docs.akash.network & https://github.com/tombeynon/akash-deploy. Thanks for the excellent guides!
* Broadly speaking you will need to:

###  Create an akash address 
``` 
make address
echo "export AKASH_ADDRESS=**address** " >> env.sh
``` 

** Fund `AKASH_ADDRESS` with at least 5 AKT
** create a deployment (you can use deploy.yaml) as an example
```
# Need to do this once
make create_certificate
```
```
make create_deployment
make list_bid
```
### accept a bid
```
make create_lease PROVIDER=**provider** DSEQ=**dseq**
```
### push the manifest
```
make send_manifest PROVIDER=**provider** DSEQ=**dseq**
```
### Read logs
```
make lease_logs PROVIDER=**provider** DSEQ=**dseq**
```

## Most importantly -- how do you interact with this?

```
make lease_status PROVIDER=**provider** DSEQ=**dseq**
```

Take a note of the URL and exposed random port. That is your ssh port to access.

```
ssh -p $RANDOMPORT root@URL
```

### Debug

If things are still not working you can set "debug=true" in the environment variable section of deploy.yaml. This will cause sshd to run with debug mode to get additional info. As a side effect as soon as you close your session it will kill the SSH session.

***

## Second Step

## Preparing your Ubuntu install

### Create your Minecraft directory and download Forge installer

```
cd /opt
```
```
mkdir minecraft
```
```
cd minecraft
```

Download your preferred Forge installer from https://files.minecraftforge.net/  Pick the Latest Installer file (not the Universal file) and copy it to your /opt/minecraft folder. 

For this example, I will use forge-1.12.2-14.23.5.2855-installer.jar which we have in our Moonbyscraft Filebase bucket, feel free to use it if you want.

```
wget https://moonbyscraft.s3.filebase.com/forge-1.12.2-14.23.5.2855-installer.jar
```

### Configure your new Modded Minecraft Server

Run the forge installer file with the --installServer flag. (change the forge version depending to the file that you have)

```
java -jar forge-1.12.2-14.23.5.2855-installer.jar --installServer
```

After the above command is complete, you will have a new file in your directory. In our case forge-1.12.2-14.23.5.2855.jar 

You can also go ahead and delete the installer.jar and installer.jar.log files

Now run the following command inside your minecraft directory (Adjust the JVM options to your needs and server.)

```
java -Xms2G -Xmx5G -jar forge-1.12.2-14.23.5.2855.jar nogui
```

#### JVM Recommendations

The best memory setting for you to use depends a lot on whether you are running 32-bit or 64-bit Java. As a good rule of thumb, your -Xmx setting needs to be:

- 64-bit Java: 1G for vanilla Minecraft + another 1G for every 25 mods installed: so 2G or 3G should be OK for small servers and a good number of mods, very large modpacks could require 5G or 6G.

- 32-bit Java: 768M for vanilla Minecraft + another 512M for every 25 mods installed 

### Accepting the EULA text

The first time you try to run your minecraft server, it will fail, as you will be required to accept the EULA. 

The EULA text file will be created inside your minecraft directory and you can edit it with the following command: 

```
nano eula.txt
```

Change the following line to true and save the file 

```
eula=true
```

### Run the server again to generate your world.

```
java -Xms2G -Xmx5G -jar forge-1.12.2-14.23.5.2855.jar nogui
```

### Stop your server and configure your server

```
Ctrl + c
```

You now have a Minecraft vanilla server, but to have a Forge modded server you will need to do a few more things. 

You will now have new folders and files inside your minecraft directory

### Copy your mods to the newly created mods folder.

We recommend getting your mods from https://www.curseforge.com/minecraft/modpacks 

Be wary with the mods that you download from other sources. There are many scams out there, but curseforge is known for being safe and trustworty.

### Edit your server.properties file (If you want to customise your server)

On a low power system you can lower the view distance. Start with 10 for your modded server, and adjust it downward if you get some lag. If you have plenty of CPU and RAM you can increase it as well. 

### Run the server again and enjoy your modded Minecraft server

```
java -Xms2G -Xmx5G -jar forge-1.12.2-14.23.5.2855.jar nogui
```

---

## Keep your server running on the decentralised cloud 

### Create a script to start minecraft 

```
cd /opt && mkdir scripts
cd scripts
nano minecraft.sh
```

### Copy and paste the following in your minecraft.sh file (Remember to adjust your forge file and JVM options depending on what you have)

```
#!/bin/bash
cd /opt/minecraft/ && java -Xms2G -Xmx5G -jar /opt/minecraft/forge-1.12.2-14.23.5.2855.jar nogui
```

### Save the file and make it executable

```
chmod +x minecraft.sh
```

### Install Screen

```
apt install screen
```

### Add a command to /etc/rc.local to start your Minecraft server everytime the server boots up. 

```
nano /etc/rc.local
```

### Add the following command, save and exit the file. This will allow the minecraft server to start in a detached screen session when the server boots up. 

```
screen -dm -S minecraft /opt/scripts/minecraft.sh 
```

### To access your console after bootup use the following command or connect via mcrcon

```
screen -r minecraft
```

### To exit the screen session use the following command

CTRL AD

---

## Backing up your Minecraft server

It is very important to backup your server

You can accomplish this with the following commands:

```
cd /opt
tar -zcvf minecraft_backup.tar.gz minecraft
```

You can now copy the created file to another server or to an external storage.

I like to use Filebase.com

## Uploading your backups to Filebase

You will need to install awscli version 1 or 2

I will show you how to install version 2 in Linux

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install
```

### Configure your awscli with your Filebase account (You will need your access key and secret key)

```
aws configure
```

This will trigger a prompt. Enter your Access Key ID and Secret Access Key. 

You can leave the region and output format blank by simply hitting enter.

After confiugring your awscli with your Filebase account, you will need to create a bucket in your Filebase account. 

You can do this via the console (web dashboard) or by running:

```
aws --endpoint https://s3.filebase.com s3 mb s3://my-minecraft-backup-bucket
```

In your backup directory, run:

```
aws --endpoint https://s3.filebase.com s3 cp minecraft_backup.tar.gz s3://my-minecraft-backup-bucket
```

Your backup file will be uploaded to your Filebase bucket and you now have a backup of your entire Minecraft server

---

## Creating Automatic Backups

These instructions will help you to create daily automatic backups of your Minecraft server, so that you don't have to worry about remembering.

This will create a daily backup in your Akash server, it will upload the backup file to your Filebase bucket and it will delete your Akash backup file afterwards

New backups will overwrite old backups in Filebase and you will have a seven day rotational backup of your Minecraft server.

### Create your backup script

```
cd /opt/scripts
nano mcbackup.sh
```

Copy and paste the following script into your nano editor. 

Change the dest= line to point to the folder that you want your backups to be made into.

```
#!/bin/sh
####################################
#
# Backup minecraft world to a
# specified folder.
#
####################################

# What to backup. Name of minecraft folder in /opt
backup_files="minecraft"

# Specify which directory to backup to.
# Make sure you have enough space to hold your backups.
# Warning: Minecraft worlds can get fairly large so choose your Akash deploy storage accordingly.
dest="/home/username/minecraftbackups"

# Create backup archive filename.
day=$(date +%A)
archive_file="$day-$backup_files-.tar.gz"

# Backup the files using tar.
cd /opt && tar zcvf $dest/$archive_file $backup_files
```

Save the file by pressing CTRL-X and entering Y 

Make the file executable

```
chmod +x mcbackup.sh
```

Test your script before creating the scheduled task and ensure that your script works 

```
/opt/scripts/mcbackup.sh
```
You should see the backup happening. Once completed, verify that the file was created. It should be in the location that you specified.

### Create your Filebase Sync script

```
cd /opt/scripts
nano filebasesync.sh
```

In your nano editor add the following, save and exit

```
# Sync minecraftbackups folder with your Minecraft backup Filebase bucket
aws --endpoint https://s3.filebase.com s3 sync /home/minecraftbackups/ s3://my-minecraft-backup-bucket
```

Make the file executable

```
chmod +x filebasesync.sh
```

### Create your Clean Minecraft Backup Folder script

```
cd /opt/scripts
nano clean-mcbackups.sh
```

In your nano editor add the following, save and exit

```
# Clean minecraftbackups after syncing to Filebase
rm -rf /home/minecraftbackups/*
```

Make the file executable

```
chmod +x clean-mcbackups.sh
```

After confirming that your scripts work, create the scheduled tasks to automate the backups.

Install Cron

```
apt install cron
``` 

- Create a schedule

- Create a scheduled task with the cron scheduler 

- Make sure that you are running this with the user that you log in to your Akash server or with root

```
crontab -e
```

If this is your first time running cron, select your preferred editor (I prefer nano)

Enter these lines at the end of your crontab and then save it.

```
01 6 * * * sudo /opt/scripts/mcbackup.sh &> /dev/null
02 7 * * * sudo /opt/scripts/filebasesync.sh
03 8 * * * sudo /opt/scripts/clean-mcbackups.sh
``` 

#### Congratulations!! You now have a Forge and modded Minecraft server running on Akash, which automatically backups to Filebase every day. 
