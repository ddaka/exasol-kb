---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Fix v8 installation fails with drive handler"
summary: "For some reason, the v8 installation can fail with the drive handler finding duplicated drives."
---
# Fix v8 installation fails with drive handler

## Description

For some reason, the v8 installation can fail with the drive handler finding duplicated drives.

## Diagnosis

Go to the `exainit.log`, which is located in `.ccc/play/local/NODE_ID/main/NODE/data/logs/cored/`.
Inside, at the end you find
```
2024-04-19 10:36:38.571673 +00:00] stage2: Run service distribution
[2024-04-19 10:36:38.571747 +00:00] stage2: Run service node_options
[2024-04-19 10:36:38.572266 +00:00] stage2: Run service prepare_update
[2024-04-19 10:36:38.572296 +00:00] stage2: Run service init_dev_structure
init_dev_structure: checking disk exa-n11-disk1
Traceback (most recent call last):
File "/opt/exasol/cos-8.45.0/bin/exainit", line 8, in <module>
sys.exit(main())
File "/opt/exasol/cos-8.45.0/lib/python3.10/site-packages/exainit/main.py", line 119, in main
run(sys.argv[1])
File "/opt/exasol/cos-8.45.0/lib/python3.10/site-packages/exainit/main.py", line 87, in run
serv.run()
File "/opt/exasol/cos-8.45.0/lib/python3.10/site-packages/exainit/tasks/init_dev_structure.py", line 62, in run
dh_inst().relink(disk_conf.devices, self.config.exaconf.container_root, self.config.exaconf.device_pool_dir, self.config.exaconf.storage_dir)
File "/opt/exasol/cos-8.45.0/lib/python3.10/site-packages/libconfd/drive_handler/drive_handler.py", line 644, in instance
__handler = drive_handler()
File "/opt/exasol/cos-8.45.0/lib/python3.10/site-packages/libconfd/drive_handler/drive_handler.py", line 64, in _init_
super(drive_handler, self)._init_(verbose)
File "/opt/exasol/cos-8.45.0/lib/python3.10/site-packages/libconfd/drive_handler/impl.py", line 150, in _init_
self._rescan()
File "/opt/exasol/cos-8.45.0/lib/python3.10/site-packages/libconfd/drive_handler/impl.py", line 161, in _rescan
self._scan_devfiles()
File "/opt/exasol/cos-8.45.0/lib/python3.10/site-packages/libconfd/drive_handler/impl.py", line 218, in _scan_devfiles
raise drive_handler_exception('duplicated devfile_id: {}'.format(devfile_id))
libconfd.drive_handler.impl.drive_handler_exception: duplicated devfile_id: drive.0x64cd98f075d0c200
```
If you have this kind of error message you ara affected by a known bug.

## How to fix

In order to fix this, execute the following script as superuser on all nodes inside the home folder of the installation user. Where (`.ccc` folder is)
```
A=$(find . -path '*libconfd/drive_handler/impl.py')
mkdir patch.backup
cp "${A}" patch.backup/impl.py
sed -i 's/EMULATION_LIST = \[\]/EMULATION_LIST = \[""\]/' "${A}"
```
Then restart the `c4_cloud_command` service
```
systemctl restart c4_cloud_command
```

Afterwards, your installation should continue normally.


