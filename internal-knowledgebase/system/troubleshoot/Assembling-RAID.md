---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Fixing Nodes Not Booting Up - Manual RAID Assembly"
summary: "This knowledge base article demonstrates how to fix nodes that are not booting up by manually assembling the RAID after a faulty disk has been replaced. This procedure is mainly..."
---
# Fixing Nodes Not Booting Up - Manual RAID Assembly

This knowledge base article demonstrates how to fix nodes that are not booting up by manually assembling the RAID after a faulty disk has been replaced. This procedure is mainly applicable to Webtrekk environments.
## Procedure

1. **Check the Node:**
   - Access the shell of the node that is not booting up (e.g., N11): `rssh n11`
2. **Check Logs:**
   - Navigate to the logs directory: `cd /var/logs/`
   - List the files: `ls -l`
   - Check the content of hddmount.log: `cat hddmount.log`
3. **Manually Assemble RAID:**
   - If you see error messages like "no recognizable superblock" or "disk OS mount failed," try manually assembling the RAID.
4. **Check hddmount_gpt.sh:**
   - From the management node, check the hddmount_gpt.sh script:
     ```bash
     grep mdadm /usr/opt/EXASuite-7/EXAClusterOS-7.1.11/var/exaoperation/cluster1/nodes/n0011/hddmount_gpt.sh
5.  **Below is the Expected Output of the above command:**
   ```bash
   UUID=$(mdadm --examine /dev/exad3p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad5p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad7p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad4p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad6p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C 2>/dev/null | awk '/^ *UUID/{u[$3]+=1;} /^ *Array UUID/{u[$4]+=1;} END{for(i in u){if(u[i]>a){x=i;a=u[i];};};print x;}')
   mdadm --assemble /dev/md1 --uuid "$UUID" -R --auto=yes /dev/exad3p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad5p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad7p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad4p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad6p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C || error 'Disk d00_os mount failed.'
## Manual RAID Assembly - Node Recovery

6. **Return to the Node:**
   - Go back to the node that is not booting: `rssh n11`
   - Execute the following command to assemble the RAID for the first disk:
     ```bash
     UUID=$(mdadm --examine /dev/exad3p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad5p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad7p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad4p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad6p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C 2>/dev/null | awk '/^ *UUID/{u[$3]+=1;} /^ *Array UUID/{u[$4]+=1;} END{for(i in u){if(u[i]>a){x=i;a=u[i];};};print x;}')
     mdadm --assemble /dev/md1 --uuid "$UUID" -R --auto=yes /dev/exad3p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad5p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad7p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad4p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C /dev/exad6p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C
     ```

7. **Disk Replacement:**
   - The disk that was replaced will most likely fail. Add it manually again:
     ```bash
     mdadm /dev/md1 --manage --add /dev/exad3p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C
     # Additional mdadm commands for other disks
     ```

8. **Check Disk Status:**
   - Inspect the status of the disk:
     ```bash
     mdadm --examine /dev/exad3p2_B97EDD414464613F552BECAF11C8FB6A3CE2BB0C
     # Repeat for other disks (md2 to md4)
     ```

9. **Reboot:**
   - After adding disks manually and verifying their status, reboot the system.

10. **Verify RAID Status:**
    - Check the status of the RAID disks:
      ```bash
      mdadm -D /dev/md0
      ```

11. **Recovery Status:**
    - Check the recovery status:
      ```bash
      cat /proc/mdstat
      ```
