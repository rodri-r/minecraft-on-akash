### This guide will allow you to run a Minecraft Paper server with plugins on Akash

## If you follow this guide, you can accomplish the following:

* Deploy an Ubuntu Docker image in Akash with CLI v0.14.0
* Setup a Paper Minecraft server
* Configure mcrcon
* Setup automatic backups to Filebase.com
* Buy me a coffee to keep the work going

| | |
--- | --- 
|akash:|`akash178d5wcg95z2jc5au608a9scshvwy3stwt3cgmq`|

***

## Deploy your Ubuntu instance on Akash

* This assumes that you have installed akash in `$HOME/bin/akash`

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
***

Take a note of the URL and exposed random port. You will use this to access the deployment shell, your Minecraft server and for remote controlling your server with MCRcon

For this guide, I use Akash CLI v0.14.0 (https://docs.akash.network/release-notes/v0.14.0)

I logged in to my Ubuntu image with:

```
akash provider lease-shell --from $AKASH_KEY_NAME --dseq yourdseq --tty --provider=$AKASH_PROVIDER web /bin/sh
```

Use the deploy.yaml file in this directory and the Deployment Guides at docs.akash.network to get your Ubuntu instance deployed on Akash.

If you prefer, you can use your own Ubuntu Docker image. 

The one in this deploy.yaml file will run Ubuntu 20.04 and it includes Openjdk-16

***

## Second Step

## Preparing your Ubuntu install

### Create your Minecraft directory and download Paper installer

```
cd /opt
```
```
mkdir minecraft
```
```
cd minecraft
```
### To keep things a little more organized, create the following diretories

```
mkdir scripts server tools
```

```
cd server
```

Download your preferred Paper MC version installer from https://papermc.io/downloads

For this example, I will use paper-1.17.1-324.jar

```
wget https://papermc.io/api/v2/projects/paper/versions/1.17.1/builds/324/downloads/paper-1.17.1-324.jar
```
### Install and Configure your Minecraft Paper server

Run the Paper jar file. (change the Paper version depending on the file that you have)

```
java -Xms2G -Xmx5G -jar paper-1.17.1-324.jar nogui
```
#### JVM Recommendations

The best memory setting for you to use depends a lot on whether you are running 32-bit or 64-bit Java. As a good rule of thumb, your -Xmx setting needs to be:

- 64-bit Java: 1G for vanilla Minecraft + another 1G for every 25 plugins installed: so 2G or 3G should be OK for small servers and a good number of plugins, very large servers could require 5G or 6G.

- 32-bit Java: 768M for vanilla Minecraft + another 512M for every 25 plugins installed 

After running the last command, you will have more files in your server directory and you will have to accept the EULA text.

### Accepting the EULA text

The first time you try to run your minecraft server, it will fail, as you will be required to accept the EULA. 

The EULA text file will be created inside your server directory and you can edit it with the following command: 

```
nano eula.txt
```

Change the following line to true and save the file 

```
eula=true
```

### Run the server again to generate your world.

```
java -Xms2G -Xmx5G -jar paper-1.17.1-324.jar nogui
```

### Stop your server and configure your server

```
Ctrl + c
```

You now have a Minecraft Paper server, but to have a Paper server with plugins, you will need to do a few more things. 

You will now have new folders and files inside your server directory

### Download your plugins to the newly created plugins folder.

There are many plugins for Minecraft servers. 

One of the advantages of a Minecraft Paper server (besides being more efficient), is that it accepts both Bukkit and Spigot plugins.

I recommend getting your plugins from:

* https://www.spigotmc.org/
* https://dev.bukkit.org/bukkit-plugins

### Edit your server.properties file (If you want to customise your server)

On a low power system you can lower the view distance. Start with 10 for your server and adjust it downward if you get some lag. If you have plenty of CPU and RAM you can increase it as well. 

### Run the server again and enjoy your Minecraft Paper server

```
java -Xms2G -Xmx5G -jar paper-1.17.1-324.jar nogui
```

***

## Keep your server running on the decentralised cloud 

### Create a script to start Minecraft

```
cd /opt/minecraft/scripts
nano minecraft.sh 
```

### Copy and paste the following in your minecraft.sh file (Remember to adjust your Paper file and JVM options depending on what you have)

```
#!/bin/bash
cd /opt/minecraft/server && nohup java -Xms2G -Xmx5G -jar /opt/minecraft/server/paper-1.17.1-324.jar nogui &
```

### Save the file and make it executable

```
chmod +x minecraft.sh
```

### Add a command to /etc/rc.local to start your Minecraft server everytime the server boots up. 

```
nano /etc/rc.local
```

### Add the following command, save and exit the file. This will allow the minecraft server to start in the background  when the server boots up. 

```
nohup java -Xms2G -Xmx5G -jar /opt/minecraft/server/paper-1.17.1-324.jar nogui &
```

## Backing up your Minecraft server

It is very important to backup your server

The following steps will allow you to automatically:

* Backup your Minecraft server on Akash on a daily basis
* Upload the backup file to Filebase (You can also take the backup file and download it to your local machine with these instructions: https://docs.akash.network/release-notes/v0.14.0#copy-file-from-akash-container-deployment
* Delete the backup file on your Akash instance

You can accomplish auto backups to Filebase.com with the following steps & commands:

### Install MCRcon

```
cd /opt/minecraft/tools
git clone https://github.com/Tiiffi/mcrcon.git
cd mcrcon
make
make install
```

You can read more about MCrcon at https://github.com/Tiiffi/mcrcon

### Create a script to stop Minecraft
 
```
cd /opt/minecraft/scripts
nano stopmc.sh
```

### Copy the following into that file

```
#!/bin/sh
####################################
##
# Stop Minecraft with MCRcon
# 
##
####################################

mcrcon -H yourserver.com -P your-rconport -p yourpassword -w 10 "say Server is restarting in 10 seconds!" save-all stop
```

Save the file by pressing CTRL-X and entering Y

### Make the file executable

```
chmod +x stopmc.sh
```

### Create your backup script

```
cd /opt/minecraft/scripts
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
/opt/minecraft/scripts/mcbackup.sh
```
You should see the backup happening. Once completed, verify that the file was created. It should be in the location that you specified.

## Uploading your backups to Filebase

You will need to install awscli version 1 or 2

I will show you how to install version 2 in Linux

```
cd /opt
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
apt install unzip
unzip awscliv2.zip
./aws/install
```

### Configure your awscli with your Filebase account (You will need your access key and secret key)

```
aws configure
```

This will trigger a prompt. Enter your Access Key ID and Secret Access Key. 

You can leave the region and output format blank by simply hitting enter.

After configuring your awscli with your Filebase account, you will need to create a bucket in your Filebase account. 

You can do this via the console (web dashboard) or by running:

```
aws --endpoint https://s3.filebase.com s3 mb s3://my-minecraft-backup-bucket
```

### Create your Filebase Sync script

```
cd /opt/minecraft/scripts
nano filebasesync.sh
```

In your nano editor add the following, save and exit

```
#!/bin/sh
####################################
#
# Sync minecraftbackups folder with your Minecraft backup Filebase bucket
#
###################################
aws --endpoint https://s3.filebase.com s3 sync /home/minecraftbackups/ s3://my-minecraft-backup-bucket
```
Save the file by pressing CTRL-X and entering Y

Make the file executable

```
chmod +x filebasesync.sh
```

### Create your Clean Minecraft Backup Folder script

```
cd /opt/minecraft/scripts
nano clean-mcbackups.sh
```

In your nano editor add the following, save and exit

```
#!/bin/sh
####################################
#
# Clean minecraftbackups after syncing to Filebase
#
###################################
rm -rf /home/minecraftbackups/*
```
Save the file by pressing CTRL-X and entering Y

Make the file executable

```
chmod +x clean-mcbackups.sh
```

### After confirming that your scripts work, create the scheduled tasks to automate the backups.

## Install Cron

```
apt install cron
```
Make sure that Cron is running 

```
service cron status
service cron start
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
01 06 * * * sudo /opt/minecraft/scripts/stopmc.sh
02 06 * * * sudo /opt/minecraft/scripts/mcbackup.sh
04 06 * * * sudo /opt/minecraft/scripts/minecraft.sh
05 06 * * * sudo /opt/minecraft/scripts/filebasesync.sh
10 06 * * * sudo /opt/minecraft/scripts/clean-mcbackups.sh
``` 

#### Congratulations!! You now have an Ubuntu instance running a Minecraft Paper server with plugins on Akash, which automatically backups to Filebase every day. 

