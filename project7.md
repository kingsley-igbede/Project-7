# Kingsley Documentation of Project 7

In this project, we will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

## Step 1 - Prepare The NFS Server

1. Spin up a new EC2 instance with RHEL Linux 8 Operating System

2. Update Server

`sudo yum update -y`

3. Create 3 volumes in the same AZ as your Project7-nfs-server EC2, each of 10 GB

![Volumes Created](./images/volumes-created.jpg)

4. Attach all three volumes one by one to Project7-nfs-server  EC2 instance

5. Use `lsblk` command to inspect what block devices are attached to the server.

![Volumes Attached](./images/volumes-attached.jpg)

6.  Use gdisk utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/nvme1n1`

![Disk 1 Partition](./images/disk1-partition.jpg)

`sudo gdisk /dev/nvme2n1`

![Disk 2 Partition](./images/disk2-partition.jpg)

`sudo gdisk /dev/nvme3n1`

![Disk 3 Partition](./images/disk3-partition.jpg)

7. Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

![Partition Status](./images/partition-status.jpg)

8. Install lvm2 package using `sudo yum install lvm2` . Run `sudo lvmdiskscan` command to check for available partitions.

`sudo yum install lvm2 -y`

![LVM2 Package Install Status](./images/lvm2-package-install-status.jpg)

`sudo lvmdiskscan`

![Partition Scan](./images/partition-scan.jpg)

`lsblk`

![List Blocks](./images/list-blocks.jpg)

9. Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM. That is create physical volumes from each of the partitions.

`sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

![Physical Volumes](./images/physical-volumes.jpg)

`sudo lvmdiskscan`

![PV scan/status](./images/pv-scan-status.jpg)

10. Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

`sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

![Volume Group Created](./images/volume-group-created.jpg)

`sudo vgs`

![Volume Group Status](./images/volume-group-status.jpg)

11. Use `lvcreate` utility to create 3 logical volumes.  lv-opt lv-apps, and lv-logs.

`sudo lvcreate -L 9G -n lv-opt webdata-vg`

`sudo lvcreate -L 9G -n lv-apps webdata-vg`

`sudo lvcreate -L 9G -n lv-logs webdata-vg`

12. Verify that your Logical Volume has been created successfully by running 

`sudo lvs`

![Logical Volumes Status](./images/logical-volumes-status.jpg)

`lsblk`

![List LV Blocks](./images/list-lv-blocks.jpg)

13. Verify the entire setup

`sudo vgdisplay -v`

![Complete Setup Status1](./images/complete-setup-status1.jpg)

![Complete Setup Status2](./images/complete-setup-status2.jpg)

![Complete Setup Status3](./images/complete-setup-status3.jpg)

14. Use mkfs to format the logical volumes with xfs filesystem

`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`

![Filesystem lvapps](./images/filesystem-lv-apps.jpg)

`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`

![Filesystem lvlogs](./images/filesystem-lv-logs.jpg)

`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

![Filesystem lvopt](./images/filesystem-lv-opt.jpg)

15. Create mount points on /mnt directory for the logical volumes as follow:

`sudo mkdir /mnt/apps`

`sudo mkdir /mnt/logs`

`sudo mkdir /mnt/opt`

`ls /mnt/`

![Mount Directories Status](./images/mount-directories-created.jpg)

*Mount lv-apps on /mnt/apps – To be used by webservers*

*Mount lv-logs on /mnt/logs – To be used by webserver logs*

*Mount lv-opt on /mnt/opt – To be used by Jenkins server*

16. Mount logical volumes on the paths

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`

`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`

`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`

17. Install NFS server, configure it to start on reboot and make sure it is up and running

`sudo yum -y update`

`sudo yum install nfs-utils -y`

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`

![NFS Server Status](./images/nfs-server-status.jpg)

18. Make sure you set up permission that will allow the Web servers to read, write and execute files on NFS.

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`

`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/logs`

`sudo chmod -R 777 /mnt/opt`

`sudo systemctl restart nfs-server.service`

19. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

`rpcinfo -p | grep nfs`

![NFS ports](./images/NFS-ports.jpg)

*In order for NFS server to be accessible from the client, you must also open the following ports: TCP 111, UDP 111, UDP 2049*

![NFS security groups](./images/nfs-security-groups.jpg)

20. Configure access to NFS for clients within the same subnet

`sudo vi /etc/exports`

![NFS cidr editor](./images/nfs-cidr-editor.jpg)

`sudo exportfs -arv`

![NFS exports](./images/nfs-exports.jpg)

21. Restart NFS Server

`sudo systemctl restart nfs-server.service`


## Step 2 - Configure The Database Server

1. Install MySQL server

`sudo apt update -y`

`sudo apt upgrade -y`

`sudo apt install mysql-server -y`

![DB Mysql Status](./images/db-mysql-status.jpg)

2. Create a database and name it **tooling**

`sudo mysql`

`create database tooling;`

`show databases;`

![Show Databases](./images/show-databases.jpg)

3. Create a database user and name it **webaccess**

`create user 'webaccess'@'172.31.32.0/20' identified by 'password';`

`grant all privileges on tooling.* to 'webaccess'@'172.31.32.0/20';`

`flush privileges;`

`exit`

![Databases User Status](./images/db-user-status.jpg)


## Step 3 - Prepare the Web Servers

- [x] From the shared storage solutions, (NFS server and MySQL database), make sure the web servers can serve the same content.
- [ ] For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

### During the next steps we will do following:

- [x] Configure NFS client (this step must be done on all three servers)
- [ ] Deploy a Tooling application to our Web Servers into a shared NFS folder
- [ ] Configure the Web Servers to work with a single MySQL database

1. Launch 3 new EC2 instances with RHEL 8 Operating System for the Webservers properly labelled

2. Update all servers

`sudo yum update -y`

3. Install NFS client on all Web Servers

`sudo yum install nfs-utils nfs4-acl-tools -y`

![Webserver1 NFS Utils](./images/webserver1-nfs-utils.jpg)

![Webserver2 NFS Utils](./images/webserver2-nfs-utils.jpg)

![Webserver3 NFS Utils](./images/webserver3-nfs-utils.jpg)

4. Mount /var/www/ and target the NFS server’s export for apps

*First create the folder /var/www/ on all webservers

`sudo mkdir /var/www`

`sudo mount -t nfs -o rw,nosuid 172.31.36.253:/mnt/apps /var/www`

5. Verify that NFS was mounted successfully by running `df -h`. Make sure that the changes will persist on Web Server after reboot:

`df -h`

![Webserver1 Mount Status](./images/webserver1-mount-status.jpg)

![Webserver2 Mount Status](./images/webserver2-mount-status.jpg)

![Webserver3 Mount Status](./images/webserver3-mount-status.jpg)

