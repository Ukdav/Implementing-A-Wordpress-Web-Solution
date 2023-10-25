# Implementing-A-Wordpress-Web-Solution-With-LVM-Storage-Mgt

A WordPress web solution with LVM (Logical Volume Management) storage management offers increased flexibility, scalability, and reliability for your website. LVM allows you to efficiently manage disk space, making adapting to changing storage needs easier. Here's a short note on how to implement this solution:

Prerequisites:

**A Linux server:** Choose a Linux distribution of your preference (e.g., Ubuntu or CentOS) and ensure it is up and running.

**LVM installed:** You must make sure the LVM package is installed on your server.

## STEP1 Preparing Web Server

* Create an EC2 instance server on AWS

* On the EBS console, create (3) three storage volumes for the instance. This serves as additional external storage to our EC2 machine

![EBS CREATION](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/0b3bb2ec-3325-4a81-9eb6-01f8a5c4d23f)

![creating vol aaaand renaming](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/c1d5a2b1-f63f-4112-bfb4-71106da16e03)

* Attach the created volumes to the EC2 instance

![attaching vol](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/08083980-ba15-441a-9657-dcea23f5fb51)

![volume attached](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/f9d0c055-3e25-4857-952d-9df0c6ded923)

* SSH into the instance and on the EC2 terminal, view the disks attached to the instance. This is achieved using the *lsblk*  command.

![lsblk](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/72c4fa29-3c80-4fb4-8274-120628da539d)

In Red Hat Enterprise Linux and other Linux distributions, nvme1n1 represents an NVMe (Non-Volatile Memory Express) device. NVMe devices are typically used for high-speed storage and are commonly found in modern servers and workstations.

All devices in linux reside in */dev/* directory. we can inspect or check it using this command *ls /dev* and make sure you see all the newly created block devices their names will likely be nvme1n1, nvme2n1, nvme3n1:

![ls dev](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/074d7312-8533-4e7d-957a-b22332118429)

* Use *df -f* command to see all mounts and free space on your server

![df -h](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/a36f1202-7ebe-46f9-b1d6-4ff813f6fde4

* Use gdisk utility to create a single partition on each of the 3 disk using the commands below:

1. *sudo gdisk /dev/nvme1n1*

2. *sudo gdisk /dev/nvme2n1*

3. *sudo gdisk /dev/nvme3n1*

![new partition 3](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/6c5dba8e-bf9a-4eff-a926-d10766c4a154)

![lsblk for new partion](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/9612c4f0-9ca7-4180-a00f-15c5b4122246)

* Installing LVM2 package for creating logical volumes on a linux server, using this command: *sudo yum install lvm2* and RUN *sudo pvs*

![sudo yum install lvm2](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/04d2af20-1b69-4a6a-8bd4-3c1a91f46644)

* Creating Physical Volumes on the partitioned disk volumes
*sudo pvcreate <partition_path>*

* Verify that your physical volume has been created successfully by running this command : *sudo pvs*

![pvcreate](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/12ff974e-1fa2-4519-9cec-9ebab549d13b)

* Use Vgcreate utility to add all 3 PVS to a volume group (VG). Name the VG webdata-vg using this command: *sudo vgcreate webdata-vg /dev/nvme1n1p9 /dev/nvme2n1p1 /dev/nvme3n1p1*
  
* verifying that your VG has been created successfully by running this commad *sudo vgs*

![vgcreate](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/2a11eb73-d3ab-47e3-8847-c177f3ab32a2)

* Use Lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size) and logs-lv use the remaining space of the PV size.

*NOTE* apps-lv will be used to store data of the website while, logs-lv will be used to store data for logs. below are the commands:

*sudo lvcreate -n apps-lv -L 14G webdata-vg*

*sudo lvcreate -n logs-lv -L 14G webdata-vg*

* Verify that your logical volume has been created successfully by running *sudo lvs*

![lvs](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/0307c237-c5ab-4622-bf83-91ebf40c1a42)

* To verify the entire setup using the below command

*sudo vgdisplay -v #view complete setup - VG, PV, and LV*

![vgdisplay](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/0c68456b-b85b-4b6c-b650-818e0bc1f612)

*Sudo lsblk*

![sudo lsblk](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/c37d3b90-3571-4a7e-952b-1d3592974efa)

* Our logical volumes are ready to be used as filesystems for storing application and log data.
* Creating filesystems on both logical volumes
  
* Use *mkfs.ext4* to formant the logical volumes with ext4 filessystem by using the below code:

*sudo mkfs -t ext4 /dev/webdata-vg/apps-lv*
*sudo mkfs -t ext4 /dev/webdata-vg/logs-lv*

* The Apache webserver uses the html folder in the var directory to store web content. We create this directory and also a directory for collecting log data of our application

* Create /var/www/html directory to store website files

*sudo mkdir -p /var/www/html/*

* Create /home/recovery/logs to store backup of log data

*sudo mkdir -p /home/recovery/logs/*

* For our filesystem to be used by the server we mount it on the apache directory. Also, we mount the logs filesystem to the log directory

* Mount /var/www/html on apps-lv logical volume

*sudo mount /dev/webdata-vg/apps-lv /var/www/html/*

![mkfs](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/2fb015ae-3a89-49e5-a5ee-60b2e615d6be)

* Use the *rsync* utility to backup all the files in the log directory /var/log/ into /home/recovery/logs (This is required before mounting the file system)

* Using the following code: *sudo rsync -av /var/log/. /home/recovery/logs/*

![rsync](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/fc3722cc-586f-4e9e-933f-9ab5c81ad746)

* Mount /var/log/ on logs-lv logical volume. (Note that all the existing data on /var/log/ will be deleted. That is why the above step is very important). 

* use this command *sudo mount dev/webdata-vg/logs-lv /var/log/*

* Restore logfiles back into /var/log/ directory using this command: *sudo rsync -av /home/recovery/logs/ /var/log*
  
![sudo rsync](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/8d964fbd-4d07-4846-a550-cb5dc6e0d23b)

* Update /etc/fstab/ file so that the mount configuration will persist after restart of the server.

The UUID of the device will be used to update the etc/fstab/ file: 

command: *sudo blkid*

![sudo blkid](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/1a415f4d-c67c-4c65-a518-2617892448fc)

*sudo vi /etc/fstab*

update /etc/fstab/ in this formant using your own UUID and remember to remove the leading and ending quotes

![vi edit](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/b562e1f0-ca50-4eb6-8851-937230c8ebe1)

* Test the configuration and reload the daemon

*sudo mount -a*

*sudo systemctl daemon-reload*

![mount and reload](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/594bb735-cce9-48ac-a234-5dce045a273a)

* Verify your setup by running df -f output be displayed like this

![df -f 2](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/c8beba06-37ea-4ace-83e0-cb76537e73ff)

## Installing WordPress and configuring to use MYSQL DATABASE

**STEP 2 Prepare the Database Server**










































