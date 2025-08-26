## Challenge #1: Disk Space Full on EC2

**Scenario:**  
Created a file that fills 8GB of disk space:

```bash
fallocate -l 8G new_file 
# to create a file which will take 8 gb storage

Problem:
After modifying the EC2 volume (8GB → 29GB), the system still shows the old size.

Solution:
```bash
Install required utils:
sudo apt install cloud-guest-utils

Resize partition and filesystem:
# Note: use commands considering which virtualized storage your machine uses
```bash
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1

Verify:
```bash
df -h

Screenshots:
Include before/after images here