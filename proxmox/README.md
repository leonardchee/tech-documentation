# Proxmox Setup Instructions
Migrating Ubuntu to Proxmox: Backup & Preparation Steps

Current Setup:
I am running Ubuntu Jammy Jellyfish 22.04 LTS on an Intel NUC. My plan is to re-purpose this Intel NUC for running Proxmox bare-metal. However, I wish to retain my current Ubuntu desktop setup for use as a VM on the new Proxmox setup once that is done. 

Objective:
To create an image of my existing Ubuntu desktop installation on an Intel NUC, intending to load this image as a VM in a new Proxmox setup. One key consideration is that my current Ubuntu has a 2TB SSD that is not fully utilised. The image that is created should only be of the actual disk usage, and not the full 2TB because that will gobble up the entire SSD space once I load this VM image into the repurposed Proxmox NUC.

Summary of Steps

1. Initial Setup and USB Boot Configuration
- Booted into Ubuntu Live via a USB drive to avoid directly modifying the running system.
- Verified disk setup using `lsblk` to identify that my main Ubuntu partition was on /dev/nvme0n1p2 (1.8 TB)
- Decided to use `partclone` to image only the used data instead of the entire 2 TB disk, targeting a 77GB compressed image file.

2. Preparing the Destination for the Image
- Mounted an external drive to store the backup image:
```
# use lsblk to identify both the external drive's identifier, as well as the partition that your data is stored on, which is your target partition for cloning.
lsblk

# Create a directory to mount the external drive
sudo mkdir /mnt/backup

# Mount the external drive
sudo mount /dev/your-ext-drive-identifier /mnt/backup
```

- Resolved mounting issues by checking the drive’s mount point (/media/ubuntu/SanDskMini) and adjusting as needed.

	
3. Installing Partclone and Creating the Image

```
# Update packages and install Partclone in the live environment
sudo apt update
sudo apt install partclone

# Ran into error saying package not found. Resolved by enabling the universe repository
sudo add-apt-repository universe
sudo apt update && sudo apt install partclone
```

- Used Partclone to create the image, specifying compression and targeting only the active partition:
```
sudo partclone.ext4 -c -s /dev/nvme0n1p2 -o /mnt/backup/ubuntu_backup.img -z1

# Ran into an error "Too small or bad buffer size". Removing the "-z1" flag resolves this error

sudo partclone.ext4 -c -s /dev/nvme0n1p2 -o /mnt/backup/ubuntu_backup.img
```

	
4. Verifying the Backup Image
```
# Check the backup file's size and ensure reasonableness to original partition size

ls -lh /mnt/backup/ubuntu_backup.img

# Test the integrity of the image file to ensure it's fully readable
sudo partclone.chkimg -s /mng/backup/ubuntu_backup.img

# After verifying the backup, unmount the drive
sudo umount /mnt/backup
```


