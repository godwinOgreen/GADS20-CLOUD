# LAB: Working with Virtual Machines

## Overview

In this lab, you set up a game application—a Minecraft server.

The Minecraft server software will run on a Compute Engine instance.

You use an n1-standard-1 machine type that includes a 10-GB boot disk, 1 virtual CPU (vCPU), and 3.75 GB of RAM. This machine type runs Debian Linux by default.

To make sure there is plenty of room for the Minecraft server's world data, you also attach a high-performance 50-GB persistent solid-state drive (SSD) to the instance. This dedicated Minecraft server can support up to 50 players.

## Objectives:

In this lab, you learn how to perform the following tasks:

- 
  Customize an application server

- Install and configure necessary software

- Configure network access

- Schedule regular backups

## Steps

1. Create the VM.

```  
gcloud beta compute --project=$DEVSHELL_PROJECT_ID instances create mc-server --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=242392822251-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/trace.append,https://www.googleapis.com/auth/devstorage.read_write --tags=minecraft-server --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mc-server --create-disk=mode=rw,size=50,type=projects/$DEVSHELL_PROJECT_ID-/zones/us-central1-a/diskTypes/pd-ssd,name=minecraft-disk,device-name=minecraft-disk --reservation-affinity=any
```

2. Prepare the data disk. 

- For mc-server, click SSH to open a terminal and connect.
``` 
gcloud compute ssh mc-server
```
- Create a directory that serves as the mount point for the data disk.
``` 
sudo mkdir -p /home/minecraft
```
- Format the disk.
```   
sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk
```
- To mount the disk.
``` 
sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft
```
3.  Install and run the application.

Install the Java Runtime Environment (JRE) and the Minecraft server.

- Update the Debian repositories on the VM.
```
sudo apt-get update
```
- After the repositories are updated, to install the headless JRE.
```
sudo apt-get install -y default-jre-headless
```
- To navigate to the directory where the persistent disk is mounted.
```
cd /home/minecraft
```
- To install the wget.
```
sudo apt-get install wget
```
- If prompted to continue, type Y.

- To download the current Minecraft server JAR file (1.11.2 JAR).
```
sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
```
Initialize the Minecraft server.

- To initialize the Minecraft server.
```
sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
```
- To see the files that were created in the first initialization of the Minecraft server.
```
sudo ls -l
```
- To edit the EULA.
```
sudo nano eula.txt
```
- Change the last line of the file from `eula=false` to `eula=true`

- Press Ctrl+O, ENTER to save the file and then press Ctrl+X to exit nano.

Create a virtual terminal screen to start the Minecraft server
.
If you start the Minecraft server again now, it is tied to the life of your SSH session: that is, if you close your SSH terminal, the server is also terminated. To avoid this issue, you can use `screen`, an application that allows you to create a virtual terminal that can be "detached," becoming a background process, or "reattached," becoming a foreground process. When a virtual terminal is detached to the background, it will run whether you are logged in or not.

- To install screen.
```
sudo apt-get install -y screen
```
- To start your Minecraft server in a screen virtual terminal, run the following command: (Use the -S flag to name your terminal mcs).
```
sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui
```
Detach from the screen and close your SSH session.
- To detach the screen terminal, press Ctrl+A, Ctrl+D
- To reattach the terminal
```
sudo screen -r mcs
```
- To exit SSH terminal 
```
exit
```
4. Allow client traffic.

Create a firewall rule.
```
gcloud compute --project=qwiklabs-gcp-04-0ea7d3fda9b8 firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server
```

5. Schedule regular backups.

Create a Cloud Storage bucket.

- For mc-server, click SSH.

```
gloud compute ssh mc-server
```
- Create a globally unique bucket name, and store it in the environment variable YOUR_BUCKET_NAME

```
export YOUR_BUCKET_NAME=godwinqwikVM
```
- Verify it with echo:

```
echo $YOUR_BUCKET_NAME
```
- To create the bucket using the gsutil tool, part of the Cloud SDK

```
gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup
```

Create a backup script.

- In the mc-server SSH terminal, navigate to your home directory.

```
cd /home/minecraft
```
- To create the script.

```
sudo nano /home/minecraft/backup.sh
```
- Copy and paste the following script into the file

```
#!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'
/usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
screen -r mcs -X stuff '/save-on\n'
```
- Press Ctrl+O, ENTER to save the file, and press Ctrl+X to exit nano.

- To make the script executable

```
sudo chmod 755 /home/minecraft/backup.sh
```

Test the backup script and schedule a cron job

- In the mc-server SSH terminal, run the backup script

```
. /home/minecraft/backup.sh
```
- In the mc-server SSH terminal, open the cron table for editing.

```
sudo crontab -e
```
- When you are prompted to select an editor, type the number corresponding to nano, and press ENTER.

- At the bottom of the cron table, paste the following line.

```
0 */4 * * * /home/minecraft/backup.sh
```
`That line instructs cron to run backups every 4 hours.`

- Press Ctrl+O, ENTER to save the cron table, and press Ctrl+X to exit nano.

Review

In this lab, you created a customized virtual machine instance by installing base software (a headless JRE) and application software (a Minecraft game server). You customized the VM by attaching and preparing a high-speed SSD data disk, and you reserved a static external IP so the address would remain consistent. Then you verified availability of the gaming server online. You set up a backup system to back up the server's data to a Cloud Storage bucket, and you tested the backup system. Then you automated backups using cron.