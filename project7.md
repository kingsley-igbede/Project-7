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










