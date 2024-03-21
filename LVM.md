**##Combining the storage of multiple hard disks and mounting them to /data:**

To combine the storage of multiple hard disks (sda, sdb, sdc, sdd) and mount the combined storage to /data, you can use LVM (Logical Volume Manager) on Linux. LVM allows you to create a single logical volume that spans multiple physical volumes (hard disks in this case), providing a flexible and manageable way to use the combined storage space.

Here's a step-by-step guide on how to achieve this with LVM:

**Step 1:** Install LVM Tools
Ensure the LVM package is installed on your system. You can install it using your distribution's package manager. For Debian/Ubuntu:
```
sudo apt-get update
sudo apt-get install lvm2
```
For CentOS/RHEL/Fedora:
```
sudo yum install lvm2
```

**Step 2:** Prepare the Disks
Warning: This process will erase all existing data on the disks. Ensure you have backups of any important data.

You need to create physical volumes on each of the disks (sda, sdb, sdc, sdd). Replace sdX with the actual disk identifiers as necessary.
```
sudo pvcreate /dev/sda
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc
sudo pvcreate /dev/sdd
```

**Step 3:** Create a Volume Group
Create a volume group that combines these physical volumes. You can name the volume group anything, here it's named vgdata.
```
sudo vgcreate data /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

**Step 4:** Create a Logical Volume
Now, create a logical volume within this volume group. You can use all available space by using the -l 100%VG option. This example names the logical volume lvdata.
```
sudo lvcreate -l 100%VG -n data data
```

**Step 5:** Format the Logical Volume
Format the logical volume with your desired filesystem. This example uses ext4.
```
sudo mkfs.ext4 /dev/data/data
```

**Step 6:** Create the Mount Point
Ensure the mount point directory exists:
```
sudo mkdir -p /data
```

**Step 7:** Mount the Logical Volume
Mount the logical volume to /data:
```
sudo mount /dev/data/data /data
```

**Step 8:** Automate the Mount at Boot
To automatically mount this volume at boot, add an entry to /etc/fstab. First, get the UUID of the logical volume:
```
sudo blkid /dev/data/data
```
Edit /etc/fstab:
```
sudo nano /etc/fstab
```
Add a line with the UUID:
UUID=&lt;UUID_of_lvdata&gt; /data ext4 defaults 0 2
Replace &lt;UUID_of_lvdata&gt; with the actual UUID you got from the blkid command.

**Step 9:** Verify
Ensure everything is working by remounting all filesystems:
```
sudo mount -a
```
Check that your logical volume is correctly mounted to /data:
```
df -h /data
```
By following these steps, you've combined the storage of multiple hard disks into a single logical volume and mounted it to /data, providing a flexible and scalable storage solution.

**##Unmounting the combined storage and mounting them individually (if needed):**

To unmount the combined storage (previously set up as a single logical volume using LVM across multiple hard drives) and revert each hard drive back to being separate and independent, you'll need to reverse the LVM configuration. This involves unmounting the logical volume, removing it, and then removing the volume group and physical volumes. This process will destroy all data on these volumes, so ensure you have backed up any important data before proceeding.

**Step 1:** Unmount the Logical Volume
First, unmount the logical volume mounted at /data:
```
sudo umount /data
```

**Step 2:** Deactivate the Logical Volume
It's a good practice to deactivate the logical volume before removing it:
```
sudo lvchange -an /dev/data/data
```

**Step 3:** Remove the Logical Volume
Now, remove the logical volume:
```
sudo lvremove /dev/data/data
```
Confirm the removal when prompted.

**Step 4:** Remove the Volume Group
With the logical volume removed, you can now remove the volume group that encompasses the physical volumes:
```
sudo vgremove data
```

**Step 5:** Remove the Physical Volumes
Finally, remove the LVM setup from each hard drive:
```
sudo pvremove /dev/sda
sudo pvremove /dev/sdb
sudo pvremove /dev/sdc
sudo pvremove /dev/sdd
```

**Step 6:** (Optional) Format the Hard Drives
If you plan to use the hard drives individually, you may want to format them with a filesystem. This step will erase all existing data on the drive. Here's an example of formatting a drive as ext4:
```
sudo mkfs.ext4 /dev/sda
```
Repeat this for each drive you wish to format, replacing /dev/sda with the appropriate device identifier.

**Step 7:** Mount the Hard Drives (Optional)
If you want to mount these drives individually, create mount points and mount them:
```
sudo mkdir /mnt/disk1
sudo mount /dev/sda /mnt/disk1
```
Repeat this for each drive, with appropriate mount points and device identifiers.

**Step 8:** Update /etc/fstab (If Necessary)
If you had previously added the logical volume to /etc/fstab for automatic mounting, remember to remove or comment out that entry to prevent boot issues. Additionally, you can add new entries for each of the individual drives you want to mount automatically.

Open /etc/fstab:
```
sudo nano /etc/fstab
```
Remove or comment out the line for the logical volume and add new lines for each drive, if desired:

\# UUID=&lt;UUID_of_lvdata&gt; /data ext4 defaults 0 2
UUID=&lt;UUID_of_sda&gt; /mnt/disk1 ext4 defaults 0 2
Replace &lt;UUID_of_sda&gt; with the actual UUID of the drive, which can be obtained using sudo blkid.

By following these steps, you will have successfully dismantled the LVM setup, reverted each hard drive to an independent state, and optionally prepared them for individual use with new filesystems. Remember to back up any important data before starting this process, as it involves data-destructive operations.