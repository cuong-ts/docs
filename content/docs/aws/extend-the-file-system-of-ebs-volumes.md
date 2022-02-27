# Extend the file system of EBS volumes

## To extend the file system of EBS volumes

1. Check disk 

```bash
[root@ip-10-69-13-54 ~]# lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  100G  0 disk
└─xvda1 202:1    0  100G  0 part /
```

2. For volumes that have a partition, such as the volumes shown in the previous step, use the **growpart** command to extend the partition. Notice that there is a space between the device name and the partition number.

```bash
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
```



