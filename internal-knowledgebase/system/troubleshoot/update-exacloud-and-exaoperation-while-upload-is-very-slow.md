---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Update EXACloud and EXAOperation while upload is very slow"
summary: "The traditional method of update is taking too much time during the upload process of the package and transfer the .PKG files to the License Server becomes necessary. Also,..."
---
# Update EXACloud and EXAOperation while upload is very slow

## Problem

The traditional method of update is taking too much time during the upload process of the package and transfer the .PKG files to the License Server becomes necessary. Also, exporting the X from the License Server does not work as per limitation on networking.

## Procedure

Update using a non-gui browser:

When you have difficulties to export the X and on remote is very slow that is actually not suitable this solution may work

0. Use Lynx instead of firefox / chrome / chromium...

 Bear in mind, you will have to deal with certificate issues, just press (Y) to ignore the warnings.

1. Stop the database and Storage

2. Transfer the patch to License Server and place it on a local drive (p.e. /tmp)

3. Launch Lynx (Also installed on the License Server is Links)

```
$ lynx https://localhost
```
4. Login using an "Administrator" account

5. Go to Software "tab" (use Down Arrow)

6. Fill in space with the file name including the path to upload manually. Then "Down Arrow".

7. Submit (Right Arrow or Return)
Note: Wait a reasonable time and check the following logs:

```
tail -f /var/log/logd/EXAoperation.log
tail -f /var/log/exaopnodestart/[NODE_IP]_start.log
tail -f /var/log/yum.log
```

8. Go to Nodes "tab"

9. Reboot License Server

10. After the License Server has come back online, log in with EXAOperation (can use your standard GUI) and check that the update has been successfully
