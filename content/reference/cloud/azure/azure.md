---
title: Portworx on Azure
weight: 1
---

This guide shows you how you can easily deploy Portworx on [**Azure**](https://azure.microsoft.com/en-us/)

### Step 1: Provision Virtual Machine
When choosing an instance, verify that you meet the [minimum requirements](/#minimum-requirements)

Portworx recommends a minimum cluster size of 3 nodes.

### Step 2: Attach Data Disk to Azure VM
Follow the instructions from Azure documentation [How to attach a data disk to a Linux VM in the Azure portal
](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-attach-disk-portal/)

Your deployment will look something like following:


![Azure Add Disk](https://docs.portworx.com/images/azure-add-disk.png "Add Disk")

### Step 3: Install Docker for the appropriate OS Version
Portworx recommends Docker 1.12 with [Device Mapper](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#/configure-docker-with-devicemapper).

Note: Portworx requires Docker to allow shared mounts.  This is standard as of Docker 1.12.  
If you are running Docker without shared mounts, please follow the steps listed [here](https://docs.portworx.com/knowledgebase/shared-mount-propagation.html)

### Step 4: Launch PX-Enterprise
[Follow the instructions to launch PX-Enterprise](/#install-with-a-container-orchestrator)

Use the docker run command to launch PX-Enterprise, substituting the appropriate multipath devices and network interfaces, as identified from the previous steps.

Alternatively, you can either run the 'px_bootstrap' script from curl, or construct your own [config.json](https://docs.portworx.com/control/config-json.html) file.

From the server node running px-enterprise container, you should see the following status:

![PX-Cluster on Azure](https://docs.portworx.com/images/azure-pxctl-status.png "PX-Cluster on Azure")


You should also be able to monitor cluster from PX-Enterprise console:

![Azure-Cluster on Lighthouse](https://docs.portworx.com/images/azure-cluster-on-lighthouse-updated.png "Azure-Cluster on Lighthouse")
