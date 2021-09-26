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

You can also go ahead and delete de installer.jar and installer.jar.log files

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
