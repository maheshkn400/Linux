Logical Volume Management (LVM) in Red Hat Linux allows for flexible disk management. Below are the steps to create, modify, and delete Physical Volumes (PVs), Volume Groups (VGs), and Logical Volumes (LVs) with examples.

### 1. Install LVM (if not already installed)

```bash
sudo yum install lvm2
```

### 2. Create Physical Volumes (PV)

To create a physical volume, you can use a disk or a partition. For example, if you have `/dev/sdb`:

```bash
sudo pvcreate /dev/sdb
```

You can check the created PVs with:

```bash
sudo pvdisplay
```

### 3. Create a Volume Group (VG)

Next, create a volume group using the physical volume you just created:

```bash
sudo vgcreate myvg /dev/sdb
```

You can check the created VGs with:

```bash
sudo vgdisplay
```

### 4. Create a Logical Volume (LV)

Now, create a logical volume within the volume group. For example, to create a 10GB logical volume named `mylv`:

```bash
sudo lvcreate -n mylv -L 10G myvg
```

You can check the created LVs with:

```bash
sudo lvdisplay
```

### 5. Format the Logical Volume

Before using the logical volume, you need to format it with a filesystem, such as ext4:

```bash
sudo mkfs.ext4 /dev/myvg/mylv
```

### 6. Mount the Logical Volume

Create a mount point and mount the logical volume:

```bash
sudo mkdir /mnt/mylv
sudo mount /dev/myvg/mylv /mnt/mylv
```

To make the mount persistent across reboots, add an entry to `/etc/fstab`:

```bash
echo '/dev/myvg/mylv /mnt/mylv ext4 defaults 0 0' | sudo tee -a /etc/fstab
```

### 7. Modify Logical Volume

To resize a logical volume, you can use the `lvresize` command. For example, to increase the size of `mylv` to 15GB:

```bash
sudo lvresize -L 15G /dev/myvg/mylv
```

NOTE: lvresize will use for both extend and reduce but lvextend only extend can't reduce with roundup the totel size


After resizing, you may need to resize the filesystem:

```bash
sudo resize2fs /dev/myvg/mylv
```

### 8. Delete Logical Volume

To delete a logical volume, use the `lvremove` command:

```bash
sudo lvremove /dev/myvg/mylv
```

### 9. Delete Volume Group

To delete a volume group, first ensure that all logical volumes within it are removed, then use:

```bash
sudo vgremove myvg
```

### 10. Delete Physical Volume

Finally, to delete a physical volume, use:

```bash
sudo pvremove /dev/sdb
```

## Certainly! Extending and reducing Volume Groups (VGs) and Logical Volumes (LVs) in LVM is a common task. Below are the steps to extend and reduce both VGs and LVs.

### Extending a Logical Volume (LV)

1. **Check Available Space in the Volume Group**

   Before extending a logical volume, check how much free space is available in the volume group:

   ```bash
   sudo vgdisplay myvg
   ```

   Look for the "Free PE / Size" line to see how much space is available.

2. **Extend the Logical Volume**

   To extend the logical volume, use the `lvextend` command. For example, to increase `mylv` by 5GB:

   ```bash
   sudo lvextend -L +5G /dev/myvg/mylv
   ```

   Alternatively, you can specify the total size:

   ```bash
   sudo lvextend -L 20G /dev/myvg/mylv
   ```

3. **Resize the Filesystem**

   After extending the logical volume, you need to resize the filesystem to use the new space. For an ext4 filesystem, use:

   ```bash
   sudo resize2fs /dev/myvg/mylv
   ```

### Reducing a Logical Volume (LV)

1. **Check Filesystem**

   Before reducing a logical volume, ensure that the filesystem is unmounted or in a read-only state. If you are reducing the size of an ext4 filesystem, you must first check and resize the filesystem:

   ```bash
   sudo umount /dev/myvg/mylv
   sudo e2fsck -f /dev/myvg/mylv
   ```

   Then, resize the filesystem to a smaller size. For example, to resize it to 10GB:

   ```bash
   sudo resize2fs /dev/myvg/mylv 10G
   ```

2. **Reduce the Logical Volume**

   Now you can reduce the logical volume. For example, to reduce `mylv` to 10GB:

   ```bash
   sudo lvreduce -L 10G /dev/myvg/mylv
   ```

3. **Mount the Logical Volume**

   If you had unmounted the filesystem, you can now mount it again:

   ```bash
   sudo mount /dev/myvg/mylv /mnt/mylv
   ```

### Extending a Volume Group (VG)

1. **Add a Physical Volume (PV)**

   To extend a volume group, you need to add a new physical volume. For example, if you have a new disk `/dev/sdc`, create a physical volume on it:

   ```bash
   sudo pvcreate /dev/sdc
   ```

   Then, add it to the volume group:

   ```bash
   sudo vgextend myvg /dev/sdc
   ```

2. **Check the Volume Group**

   You can check the updated size of the volume group:

   ```bash
   sudo vgdisplay myvg
   ```

### Reducing a Volume Group (VG)

1. **Remove a Logical Volume (LV)**

   Before reducing a volume group, ensure that all logical volumes within it are removed or moved to another volume group. For example, to remove `mylv`:

   ```bash
   sudo lvremove /dev/myvg/mylv
   ```

2. **Remove a Physical Volume (PV)**

   If you want to remove a physical volume from the volume group, first ensure that it is not in use. You can use the `vgreduce` command to remove it:

   ```bash
   sudo vgreduce myvg /dev/sdc
   ```

3. **Remove the Physical Volume**

   Finally, you can remove the physical volume:

   ```bash
   sudo pvremove /dev/sdc
   ```

### Summary of Commands

- **Create PV**: `pvcreate /dev/sdb`
- **Create VG**: `vgcreate myvg /dev/sdb`
- **Create LV**: `lvcreate -n mylv -L 10G myvg`
- **Format LV**: `mkfs.ext4 /dev/myvg/mylv`
- **Mount LV**: `mount /dev/myvg/mylv /mnt/mylv`
- **Resize LV**: `lvresize -L 15G /dev/myvg/mylv`
- **Delete LV**: `lvremove /dev/myvg/mylv`
- **Delete VG**: `vgremove myvg`
- **Delete PV**: `pvremove /dev/sdb`

- **Extend LV**: 
  ```bash
  sudo lvextend -L +5G /dev/myvg/mylv
  sudo resize2fs /dev/myvg/mylv
  ```

- **Reduce LV**: 
  ```bash
  sudo umount /dev/myvg/mylv
  sudo e2fsck -f /dev/myvg/mylv
  sudo resize2fs /dev/myvg/mylv 10G
  sudo lvreduce -L 10G /dev/myvg/mylv
  sudo mount /dev/myvg/mylv /mnt/mylv
  ```

- **Extend VG**: 
  ```bash
  sudo pvcreate /dev/sdc
  sudo vgextend myvg /dev/sdc
  ```

- **Reduce VG**: 
  ```bash
  sudo lvremove /dev/myvg/mylv
  sudo vgreduce myvg /
  ```
  
 ### Notes

- Always ensure you have backups of important data before modifying disk partitions or volumes.
- The commands may require root privileges, so use `sudo` as necessary.
- Adjust the device names and sizes according to your specific requirements.
