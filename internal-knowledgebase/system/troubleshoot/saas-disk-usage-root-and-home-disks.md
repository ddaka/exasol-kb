---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - Disk Usage (root and home disks)"
summary: "In this article, we'll go over how to bypass disk usage issues."
---
# SaaS - Disk Usage (root and home disks)

In this article, we'll go over how to bypass disk usage issues.

### Step 1

Connect via AWS Console or CLI, follow [this article](saas-login-to-sso.md)

### Step 2

If an error like No space left on device `var/log/` appears in the instance’s system log, then the file system containing the listed folder is full. (In this example, /var/log is the folder.)

### Step 3

Follow the Steps of detaching the volume:

**Note:**  If an EBS volume is the root device of an instance, you must stop the instance before you can detach the volume.

**Root Device:**

1. Stop the impaired instance
2. Open the Amazon EC2 Console
3. Select the Instance
4. Highlight the Storage Tab
5. Select the volume
6. When prompted for the information, choose Yes, Detach
7. Use Instances filter by Tag: `SAAS-1114` to Attach EBS volumes

1. Attach the EBS volume as a secondary device
2. Connect to the instance
3. Create a mount point directory (/rescue) for the new volume attached to the new instance
4. `$ sudo mkdir /rescue`
5. Mount the volume at the directory that you created in step 10.
6. `$ sudo mount /dev/xvdf1 /rescue`
7. Run the du -h command to check which files are taking up the most space
8. `$ sudo find / -type f -exec du -Sh {} + | sort -rh | head -n 10`
9. Verify that logs are uploaded in the s3 bucket before deleting them from the host
10. Find s3 bucket name filtering s3 from Cloudformation Resources
11. Click on the s3 bucket
12. Go to Folder
13. Delete the tar.gz files that match the backup in the s3 bucket from /recovery/var/lib
14. Run the unmount command
15. `$ sudo umount /rescue`
16. Detach the secondary volume from the instance. Then, attach it to the original instance as /dev/xvda (root volume)
17. Start the instance

**Non-Root Device:**

1. Run the du -h command to check which files are taking up the most space
2. `$ sudo find / -type f -exec du -Sh {} + | sort -rh | head -n 10​`
3. Verify that logs are uploaded in the s3 bucket before deleting them from the host
4. Find s3 bucket name filtering s3 from Cloudformation Resources
5. Click on the s3 bucket
6. Go to Folder
7. Delete the tar.gz files that match with s3 bucket backup logs from the host

## Additional References

**Disk Status:** <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-volume-status.html>
