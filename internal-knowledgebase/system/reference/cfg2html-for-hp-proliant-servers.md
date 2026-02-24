---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "cfg2html for HP ProLiant servers"
summary: "Sometimes HP support needs this cfg2html report, attached to this article is a working standalone version of this BASH script."
---
# cfg2html for HP ProLiant servers

Sometimes HP support needs this cfg2html report, attached to this article is a working standalone version of this BASH script.

```
mak@evo:~/cfg2html$ sudo bash cfg2html-linux221 -h

cfg2html-linux version 2.21-2011-07-18
WARNING, use this script AT YOUR OWN RISK

    Usage: cfg2html-linux221 [OPTIONS]
    creates a HTML and plain ASCII host documentation

    -o          set directory to write or use the environment
            variable OUTDIR="/path/to/dir" (directory must exist)
    -v          output version information and exit
    -h          display this help and exit
    use the following options to disable / enable collections:

    -s          disable: System
    -c          disable: Cron
    -S          disable: Software
    -f          disable: Filesystem
    -l          disable: LVM
    -L      disable: Screen tips inline
    -k          disable: Kernel/Libraries
    -e          disable: Enhancements
    -n          disable: Network
    -a          disable: Applications
    -H          disable: Hardware
    -p          enable: HP Proliant Server log files and settings
    -A          disable: Altiris ADL agent log files and settings
    -P      enable: cfg2html plugin architecture
```
