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

![attached the voume](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/f606c467-392b-41c2-b40e-1146e5b0eac9)

* SSH into the instance and on the EC2 terminal, view the disks attached to the instance. This is achieved using the *lsblk*  command.

![lsblk](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/9e3d58db-eba0-42ac-a3fc-80820c943f57)

In Red Hat Enterprise Linux and other Linux distributions, nvme1n1 represents an NVMe (Non-Volatile Memory Express) device. NVMe devices are typically used for high-speed storage and are commonly found in modern servers and workstations.

All devices in linux reside in */dev/* directory. we can inspect or check it using this command *ls /dev* and make sure you see all the newly created block devices their names will likely be nvme1n1, nvme2n1, nvme3n1:

![ls dev command](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/66716472-3d94-4b88-bdfc-3abbd267b68a)

* Use *df -f* command to see all mounts and free space on your server

![df -hcommand](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/a8d187a1-c51c-4ece-9d47-b662d0fc003a)

* Use gdisk utility to create a single partition on each of the 3 disk using the commands below:

1. *sudo gdisk /dev/nvme1n1*

2. *sudo gdisk /dev/nvme2n1*

3. *sudo gdisk /dev/nvme3n1*

![sudo gdisk](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/24bc17b7-06f0-49e9-9f81-82617f483db1)

![attaches disk attache](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/a6f87233-4921-4707-b6a5-b8e681582408)

* Installing LVM2 package for creating logical volumes on a linux server, using this command: *sudo yum install lvm2* and RUN *sudo pvs*

![sudo yum install lvm2](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/e73acaca-8436-438b-a3b7-fa7a9834ae9d)

* Creating Physical Volumes on the partitioned disk volumes
  
*sudo pvcreate <partition_path>*

sudo pvcreate /dev/nvme1n1p1

sudo pvcreate /dev/nvme2n1p1

sudo pvcreate /dev/nvme3n1p1

Verify that your physical volume has been created successfully by running this command : *sudo pvs*

![sudo  pvcreate and pvs](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/4ce8596b-9be0-4e4f-ae54-23bd9681f246)

* Use Vgcreate utility to add all 3 PVS to a volume group (VG). Name the VG webdata-vg using this command: *sudo vgcreate webdata-vg /dev/nvme1n1p9 /dev/nvme2n1p1 /dev/nvme3n1p1*
  
* verifying that your VG has been created successfully by running this commad *sudo vgs*

![sudo vgcreated and vgs](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/64f81935-36d2-432d-b3c0-cd28382fbff5)

* Use Lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size) and logs-lv use the remaining space of the PV size.

**NOTE** apps-lv will be used to store data of the website while, logs-lv will be used to store data for logs. below are the commands:

*sudo lvcreate -n apps-lv -L 14G webdata-vg*

*sudo lvcreate -n logs-lv -L 14G webdata-vg*

* Verify that your logical volume has been created successfully by running *sudo lvs* and *lvsdiskscan8

![sudo lvcreated](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/647824e5-5645-4fff-82d8-ae90b8358207)

![sudo lvmdiskscan](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/73052701-6e3b-4bac-b355-0d9e0b65cc03)

* To verify the entire setup use the below command

*sudo vgdisplay -v #view complete setup - VG, PV, and LV*

![vgdisplay](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/0c68456b-b85b-4b6c-b650-818e0bc1f612)

*Sudo lsblk*

![lsblk2](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/42ed8970-0da8-4264-ac3a-57d63c38a54c)

**Our logical volumes are ready to be used as filesystems for storing application and log data**
  
* Creating filesystems on both logical volumes
  
* Use *mkfs.ext4* to formant the logical volumes with ext4 filessystem by using the below code:

*sudo mkfs -t ext4 /dev/webdata-vg/apps-lv*
*sudo mkfs -t ext4 /dev/webdata-vg/logs-lv*

![sudo mkfs](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/fc295ece-8d5f-474c-a07e-ef6a8dabcc6d)

* The Apache webserver uses the html folder in the var directory to store web content. We created this directory and also a directory for collecting log data of our application

* Create /var/www/html directory to store website files

*sudo mkdir -p /var/www/html/*

**Create /home/recovery/logs to store backup of log data**

*sudo mkdir -p /home/recovery/logs/*

* For our filesystem to be used by the server we mount it on the apache directory. Also, we mount the logs filesystem to the log directory

* Mount /var/www/html on apps-lv logical volume

*sudo mount /dev/webdata-vg/apps-lv /var/www/html/*

![mkdir](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/11f271ef-00de-47ff-936c-0f9056912cef)

* Use the *rsync* utility to backup all the files in the log directory /var/log/ into /home/recovery/logs (This is required before mounting the file system)

* Using the following code: *sudo rsync -av /var/log/. /home/recovery/logs/*

* Mount /var/log/ on logs-lv logical volume. (Note that all the existing data on /var/log/ will be deleted. That is why the above step is very important). 

* use this command *sudo mount dev/webdata-vg/logs-lv /var/log/*

* Restore logfiles into /var/log/ directory using this command: *sudo rsync -av /home/recovery/logs/ /var/log*

![sudo moung rsync](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/3186cdd9-7a69-4c8c-9326-14ba3b09b853)

![sudo moung rsync](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/675511ba-5c88-4577-b99c-e9ad2d9cab5a)

* Update /etc/fstab/ file so that the mount configuration will persist after restart of the server.

The UUID of the device will be used to update the etc/fstab/ file: 

command: *sudo blkid*

![sudo blkid](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/5a4c87e3-740a-414c-8a74-70b3882d9155)

*sudo vi /etc/fstab*

update /etc/fstab/ in this formant using your UUID and remember to remove the leading and ending quotes

![sudo vi etcfstab](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/7218bc21-6409-454d-80d3-ff95b1812c27)

![sudo vi etcfstab1](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/c0dbe65d-80f2-42af-ac66-4438b980ed20)

**Test the configuration and reload the daemon**

*sudo mount -a*

*sudo systemctl daemon-reload*

* Verify your setup by running df -h output be displayed like this

![df hmount mapper](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/dde40441-c931-4dad-acfb-ce0403a4a51d)

## Installing WordPress and configuring to use MYSQL DATABASE

**STEP 2 Prepare the Database Server**

Repeated all the steps taken to configure the web server on the db server. Changed the apps-lv logical volume to db-lv

![df -hcommand](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/078fcf4f-a45d-4f28-be58-d7a2f89f6523)

**Step 3: Configuring Web Server**

**Run updates and install httpd on web server**
  
*yum install -y update*

![sudo yum update](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/7e81dea9-e183-48f5-b082-9a7b6db4b1e1)

*sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json*

![update all webapps](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/35fb5ecf-e977-4b51-802e-b83411f7dd5b)

**Start web server**

![mysql on dbserver](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/d5f8d445-e9c1-45ba-ba84-b26626aeaef8)

**Installing PHP and its dependencies**
  
![install php](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/8407c256-37a2-424d-9047-58658f530fcb)

![module enable](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/fb44dbd9-ec6c-458d-9957-6be3c04cbba1

![module reset](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/0114e1f5-8767-4f8b-9553-a3f0b0ff1bd5)

![update all webapps](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/a5eff0c2-1879-4c41-8ae3-4996933192c5)

![update php list](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/05a4828a-10b3-4fc3-ad61-6a41487a7652)

![utility update](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/84c627af-f0ac-423f-8166-350ba315e8df)

![fedoral project](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/3272abfa-dddb-488e-b812-fa20e4083d04

**Restarting Apache: sudo systemctl restart httpd**

**Downloading wordpress and moving it into the web content directory**

* mkdir wordpress
  
* cd   wordpress
  
*sudo wget http://wordpress.org/latest.tar.gz*
*sudo tar xzvf latest.tar.gz*
*sudo rm -rf latest.tar.gz*
*sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php*
*sudo cp -R wordpress /var/www/html/*

**Configure Selinux Policies**
* 
*sudo chown -R apache:apache /var/www/html/wordpress*
*sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R*
*sudo setsebool -P httpd_can_network_connect=1*

![copy wordpress to varwwwhtml](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/7a5d7a83-9a65-4953-874b-f3f8ef85be9c)

![configure SEluns Policies](https://github.com/Ukdav/Implementing-A-Wordpress-Web-Solution/assets/139593350/cc854d84-6c39-45e8-b26c-6bffc77f2ddb)


























































