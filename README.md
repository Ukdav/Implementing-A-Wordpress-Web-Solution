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

* SSH into the instance and on the EC2 terminal, view the disks attached to the instance. This is achieved using the -lsblk- command.

![lsblk](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/72c4fa29-3c80-4fb4-8274-120628da539d)

In Red Hat Enterprise Linux and other Linux distributions, nvme1n1 represents an NVMe (Non-Volatile Memory Express) device. NVMe devices are typically used for high-speed storage and are commonly found in modern servers and workstations.

All devices in linux reside in /dev/ directory. we can inspect or check it using this command ls /dev/ and make sure you see all the newly created block devices their names will likely be nvme1n1, nvme2n1, nvme3n1:

![ls dev](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/074d7312-8533-4e7d-957a-b22332118429)








