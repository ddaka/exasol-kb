---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "Data nodes in GCP are not booting due to Kernel Panic"
summary: "Internal knowledge base article about Data nodes in GCP are not booting due to Kernel Panic."
---
# Data nodes in GCP are not booting due to Kernel Panic

![GCP Kernel Panic](images/GCP-kernel-panic.jpg)

1. STOP (shutdown) the affected VM, click on the VM name, and Edit its properties
2. Scroll to STORAGE and then click DETACH BOOT DISK
3. Go back to VM INSTANCES and select and click a VM that is booting properly
4. Scroll to the STORAGE section and click on the BOOT DISK and then CLONE
5. Rename the cloned disk using the same naming convention (just add _new to the newly cloned disk)
6. Select LOCATION "Single zone"
7. Untick/Disable the "Snapshot schedule" and CREATE
8. Go back to the VM that is not booting and attach the newly cloned disk
9. Start the VM with the cloned disk and observe the boot process in EXAopeartion
10. Everything should work as described and the node should be running fine now
