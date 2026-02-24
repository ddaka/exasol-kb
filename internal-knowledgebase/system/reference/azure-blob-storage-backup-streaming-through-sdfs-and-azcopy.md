---
tool_name: cos
doc_type: reference
category: system
title: "Azure Blob Storage backup streaming through SDFS and azcopy"
summary: "wget [azcopy_linux_amd64_10.13.0.tar.gz](https://azcopyvnext.azureedge.net/release20211027/azcopy_linux_amd64_10.13.0.tar.gz)"
---
# Azure Blob Storage backup streaming through SDFS and azcopy

wget  [azcopy_linux_amd64_10.13.0.tar.gz](https://azcopyvnext.azureedge.net/release20211027/azcopy_linux_amd64_10.13.0.tar.gz)

azcopy login

-...-

user needs permission "Storage Blob Data Owner" for the blob container...

for i in $(sdfs shortlist v9 |grep level_0); do sdfs getraw v9 $i |./azcopy cp <https://makmak.blob.core.windows.net/backups/$i> --from-to PipeBlob;done
