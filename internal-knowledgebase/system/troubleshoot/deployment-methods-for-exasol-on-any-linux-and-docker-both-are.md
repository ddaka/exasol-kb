---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Deployment methods for Exasol on any Linux and Docker (both are using the so-called &quot;NGA&quot; architectu"
summary: "Both deployment methods for Docker and the Linux binaries are based on the next generation architecture. NGA introduces a new set of cluster partitions and new ways to deploy and..."
---
# Deployment methods for Exasol on any Linux and Docker (both are using the so-called &quot;NGA&quot; architectu

Both deployment methods for Docker and the Linux binaries are based on the next generation architecture. NGA introduces a new set of cluster partitions and new ways to deploy and configure Exasol. For an architecture overview please refer to [NG ARCHITECTURE](https://exasol.atlassian.net/wiki/spaces/RD/pages/12159415/NG+ARCHITECTURE). What makes them different to existing products like on-prem, VMs and the public clouds?

PROs:

* EXAConf config file describes the complete configuration of the Exasol cluster, services (ssh, storage, buckets), drivers and the database(s)
* No PXE required
* No mgmt. node required
* ConfD XML-RPC API (an example on how to use it can be found [here](https://exasol.my.site.com/s/article/Using-XML-RPC-to-manage-Docker-clusters))
* The first node available in COS is elected as master node
* Can run on almost any Linux (MacOS, Windows and WSL are not tested)
* Exasol does not need to ship security patches for the OS
* Exasol can be embedded in existing and pre-installed machines, we are just starting processes in an existing OS (downside: this adds complexity. Deployment can be installed and operated in any folder)
* Use standard Linux tools to control the cluster (e.g. systemctl)
* We can now use almost any block device (downside: this adds complexity)
* Deployment can be automatised using standard software
* Multiple Exasol nodes can now be operated on one host - this is extremely helpful for hosts with more than one NUMA domain (performance optimisation) as we can now pin Exasol nodes to specific resource groups inside the OS.
	+ Example: a host having one CPU socket and one CPU with multiple NUMA domains - run at least one Exasol node per NUMA domain and use Docker or cgroups to pin each Exasol node to a specific set of CPUs/threads
	+ Example: a host having two CPU socket and two CPUs with one NUMA domain per CPU - run at least host one Exasol node per CPU.
	+ Example: a host having four CPU sockets with four CPUs with one NUMA domain - run at least four Exasol nodes (one per CPU) and use cgroups to split up the memory in four groups
	+ Many more examples/combinations might be possible as this depends on how the server hardware is configured and set up for example AMD Epyc CPUs can offer their L-3 cache as NUMA domain

CONs:

* No UI (strong knowledge about many COS commands required)
* Using existing hosts add complexity (systems are no longer pre-defined)
* For Docker based setups the privileged mode is required, this allows the container to access all devices of host system
* More knowledge of the underlying OS is required - increase of Linux admin activities
* Freedom of choice regarding tools, block devices, configuration steps depending on the Linux OS and the environment
* Resource monitoring also for the hosts required -  it is not enough to monitor the Exasol containers - no host, no container
* Built-in monitoring capabilities are limited

**What is the difference between Docker and the Linux binaries?**

The Linux binaries are the extract of an Exasol Docker container. This exported TAR file includes COS, the EXARuntime and the Exasol DB binaries. At the moment this is the only way to create Exasol binaries for non-Docker environments.

A quick example on how to create those binaries:

```
docker pull exasol/docker-db:latest
docker run --name n11 -p 0.0.0.0:8899:8888 --network=bridge --detach --privileged -v /mnt/docker/:/exa exasol/docker-db:latest
docker export n11 > exasol-latest.tar
```

**When to choose which deployment method?**

It depends on what is allowed in the specific target environment e.g. if security doesn't allow custom Operating Systems and Docker then the only way to deploy Exasol is using their "OS" which is predefined by the customer/prospect. That way they have full control on the host and the installed OS, also security patches are then applied to the underlying OS.

**Any other considerations?**

In general anything that was relevant for on-prem and cloud systems is also relevant for Docker and the Linux binary deployment - the only difference: it needs to be done on the host.

* ensure the host has enough hardware resources available to run the Operating System
* Hugepages
* Exasol sysctl parameters
* OS RAM
* CPU governor
* Firewall on the host OS is allowing all Exasol related traffic
* Keep the host a simple as possible - less services is better - remember: we are looking for maximum performance any other service running on that system might influence the performance of the system

Docker host:

* Docker has many options to control/manipulate the behaviour of the container - pay attention on how things have been set up:
	+ Mount folders or files from the host inside the container
	+ Host network or bridge network
	+ the location and the type of device for the persistent data of the database should be chosen wisely (raw device, file device, LVM, mdadm, iSCSI, partitions with or without FS, FibreChannel devices, block devices with partitions or without partitions, etc... )

Linux host:

* Systemctl Services on all hosts to stop all their Exasol containers
* Update systemctl services after updating to a new Exasol release

**Documentation (as of now)**

**Docker**

* A so-called Docker image (<https://github.com/exasol/docker-db/releases>) needs to be published by Exasol and is then available through <https://hub.docker.com/r/exasol/docker-db>. Docker Hub is nothing else than a library of Docker containers. Container images might also kept in other tools like e.g. Artifactory.
* A good getting started guide can be found here: <https://github.com/exasol/docker-db>
* Articles on day-to-day tasks/operations can be found in Jira: 'project = SOL AND labels = docker'
* Or in the community: [Docker Guide](https://exasol.my.site.com/s/article/Docker-Guide)

**Linux binaries**

* Please read this article [How-to Install Exasol on any Linux (NGA)](https://exasol.atlassian.net/wiki/download/attachments/175702159/%5BSOL-687%5D%20How-to%20Install%20Exasol%20on%20any%20Linux%20(NGA).mhtml?api=v2) it describes how to install the binaries to any Linux
* The last four sections describe how to update such an deployment [How-to Install Exasol on any Linux (NGA)](https://exasol.atlassian.net/wiki/download/attachments/175702159/%5BSOL-687%5D%20How-to%20Install%20Exasol%20on%20any%20Linux%20(NGA).mhtml?api=v2)
