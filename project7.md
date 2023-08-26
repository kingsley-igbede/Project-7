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

