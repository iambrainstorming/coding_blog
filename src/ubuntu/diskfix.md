# Disk Fix

Check Disk Label and Partition: 

Ensure the disk label and partition are correctly set up. Use `sudo fdisk -l` to list all partitions and verify the correct device and partition.
```bash
sudo fdisk -l
```


```bash
sudo apt install nfs-common cifs-utils
```

```bash
sudo ntfsfix -d /dev/sdb1
```

```bash
sudo fsck /dev/sdb1
```
