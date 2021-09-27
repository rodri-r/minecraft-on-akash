### This allows you to deploy a Minecraft paper server with plugins on Akash

- Download Bukkit/Spigot plugin jars, such as EssentialsX,  and place them in the plugins directory. 
- Build your Docker image (Inside your directory, run - docker build -t name .) Or you can use ours moonbys/minecraft:paperp
- At image build time the COPY step will place those jars in /plugins. 
- At container startup, the contents of /plugins are sync'ed into /data/plugins for use with Bukkit/Spigot/Paper server types.
- Have fun & enjoy

***

### The moonbys/minecraft:paperp Docker image, runs the latest version of Minecraft and it comes with EssentialsX, Towny, Holographic Displays and SlimeFun.

#### Notes - We will be modifying this image and server soon!

