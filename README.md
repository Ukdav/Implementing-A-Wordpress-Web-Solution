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




















