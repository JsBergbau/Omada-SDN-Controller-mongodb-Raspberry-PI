# Omada SDN Controller on Raspberry PI / MongoDB 3.2.12 precompiled for Raspberry PI

There are a lot of tutorials and instructions of how to run Omada Controller on Raspberry PI. Official realeases of TP-Link are only for x86/x64 Linux.
Since Omada Controller uses Java it is quite easy to run it also on ARM / Raspberry PI.

Update 04.09.2021: At the time of writing v4.3.5 was the latest version of Omada Controller. However this tutorial works exactly the same way for latest controller v4.4.4. To update to latest controller version see section [Updating Omadacontroller to latest version](#updating-omadacontroller-to-latest-version).

Latest Omada SDN Controller v4.3.5 requires at least MongoDB 2.6. However in Debian repositories latest version is 2.4. MongoDB changed their license in Ocotober 2018, see https://www.mongodb.com/support-policy
. In June 2018 MongoDB 4.0 was released so it is a pity that Debian repositories only contain version 2.4. On the other hand version 3.2 is the last version that runs on 32 Bit, so version 3.2 should be included included
in Debian repositories. Nevertheless it is possible to run Omada SDN Controller on Raspberry PI with 32 Bit OS.

# Running Omada SDN Controller on Raspberry PI

## Installing Omada SDN Controller

First download Omada SDN Controller v4.3.5 (or newer)

Extract it to your directory of your choice, like `/opt/tplink`, no need to install it.
Omada SDN Conroller needs JDK version 8. Download it from here https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html

## Installing Java JDK

For Raspberry PI 3 with 32 Bit Raspian / Raspberry PI OS I've chosen `jdk-8u291-linux-arm32-vfp-hflt.tar.gz`

Extract it also to /opt/tplink/ so you have `/opt/tplink/jdk1.8.0_291/bin/java`

## Installing MongoDB

You could compile it yourself. However this proccess requires a lot to be installed. Instructions are here https://koenaerts.ca/compile-and-install-mongodb-on-raspberry-pi/
However these instructions are not enough. There will be a lot of errors while trying to build. To solve this there https://github.com/hinchliff/rpi2-mongodb-compile are is a lot 
of information and instruction of how to build it. 
You can Download MongoDB 3.0.14 precompiled here https://andyfelong.com/2017/08/mongodb-3-0-14-for-raspbian-stretch/
I didn't test Omada SDN Controller 4.3.5 with that version, because I've compiled version 3.2.12 and use this binary. You can download this compiled version from my repository. 

RAR: https://github.com/JsBergbau/Omada-SDN-Controller-mongodb-Raspberry-PI/blob/main/mongod.rar?raw=true

ZIP: https://github.com/JsBergbau/Omada-SDN-Controller-mongodb-Raspberry-PI/blob/main/mongod.zip?raw=true

Debugsymbols have been removed via `sudo strip --strip-debug mongod`. Still this binary is with 77 MB quite big. However this binary contains all libraries, so no need to install fitting
libraries with the correct version.

You only need `mongod`, so my compiled version only includes this executable. Put this file to

`/opt/tplink/Omada_SDN_Controller_v4.3.5_linux_x64/bin/` and make it executable via `chmod +x mongod`

## Editing Omada control script

Open `/opt/tplink/Omada_SDN_Controller_v4.3.5_linux_x64/bin/` and change 

`JRE_HOME="$( readlink -f "$( which java )" | sed "s:bin/.*$::" )"`


to 

`JRE_HOME="/opt/tplink/jdk1.8.0_291"`

In line `JAVA_OPTS=` change option `-server` to `-client`. This should require less memory. According to https://stackoverflow.com/a/198651/11890443 `-client` option is ignored on 64 Bit systems (we use 32 bit here, so just for curiosity). 

According to there https://www.javacodegeeks.com/2011/07/jvm-options-client-vs-server.html there are some differences in both VMs. 

And last change the running user to "omadacontroller".

So before `OMADA_USER=${OMADA_USER:-root}`, after `OMADA_USER=omadacontroller`

## Create user `omadacontroller`

It is very easy to add the user omadacontroller

`sudo adduser omadacontroller --shell=/bin/false --no-create-home --disabled-password`

You can just hit enter in the next dialog, no need to enter details for the user. This user has no shell and also with disabled password he is also not able to login, see https://unix.stackexchange.com/a/155151/334883

If anything went wrong, you can delete the user via `sudo userdel omadacontroller`

## Correct permissions

Now to make everything work, permissions have to be set. Go to `/opt/tplink` and execute `sudo chown -R omadacontroller *`

## Start Omada Controller

Now when everything was done correctly, Omada controller can be started via `sudo /opt/tplink/Omada_SDN_Controller_v4.3.5_linux_x64/bin/control.sh start`

## Running inside firejail

You can run OmadaController inside firejail to run it on its own IP address 

`firejail --name=omada --noprofile --net=eth0 --ip=192.168.150.254 --seccomp --nodbus --mac=aa:bb:cc:dd:ee:ff /opt/tplink/Omada_SDN_Controller_v4.3.5_linux_x64/bin/control.sh start &`

to stop it run

`firejail --join=omada /opt/tplink/Omada_SDN_Controller_v4.3.5_linux_x64/bin/control.sh stop`

Note: Using Firejail with Wifi is currently not possible, see https://github.com/netblue30/firejail/issues/3000

## Updating Omadacontroller to latest version

Do the same steps like for installing, especially editing control.sh, copying `mongod` and making both executable. Then copy everything from the old omadacontroller dir like /opt/tplink/Omada_SDN_Controller_v4.3.5_linux_x64/data/db/ to /opt/tplink/Omada_SDN_Controller_v4.4.4_linux_x64/data/db/
Check that the database files are directly copied to db directory and not another subdirectory db.

Then as last step do the [chown step](#correct-permissions) and run controller. Now the latest version is running.
