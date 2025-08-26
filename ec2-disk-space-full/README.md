# Challenge #1: Disk Space Full on EC2

**Scenario:**  
you ec2 instance disk is full.


## Step 1: Simulate the Problem (Fill the Disk)
```bash
fallocate -l 8G new_file 
```
# Check disk usage to confirm it's full
df -h


[add problem screenshot herre]

solution:
# Step 2: Expand the EBS Volume in the AWS Consolego to instance and modify volume(modify it from 8gb to 29gb)
1. Go to the EC2 Dashboard > Volumes.
2. Find the volume attached to your instance.
3. Select the volume and click Actions > Modify.
4. Increase the size (e.g., from 8 GiB to 29 GiB).

Click Modify.
[add ec2 volume screenshot here]

Problem:
After modifying the EC2 volume (8GB → 29GB), the system still shows the old size.

Solution:
```bash
Install required utils:
sudo apt install cloud-guest-utils
```
# Step 3: Expand the Partition and Filesystem on the Instance
Modifying the volume in AWS only expands the hardware. You must now expand the software (partition and filesystem) to use the new space.

Resize partition and filesystem:
## Note: use commands considering which virtualized storage your machine uses
For xvda systems:
```bash
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
```
For nvme systems:  
```bash
sudo growpart /dev/nvme0n1 1
sudo resize2fs /dev/nvme0n1p1
```


Verify:
```bash
df -h
```
<img width="965" height="471" alt="Screenshot 2025-08-26 200645" src="https://github.com/user-attachments/assets/ea43ae58-2a9f-4caa-bdde-60ad9f089606" />


📚 Key Takeaways
- Always check lsblk before running growpart to confirm your device name (nvme0n1, xvda).
- cloud-guest-utils contains the essential growpart command.
- The two-step process: 1. growpart (resizes partition), 2. resize2fs (resizes filesystem).
- Modifying the volume in the AWS console does not automatically resize the OS filesystem.
- Problem Solved! Your instance now has the expanded storage available.
