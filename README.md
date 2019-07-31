# terraform-libvirt-kubernetes

A small project describing how to use [terraform](https://www.terraform.io/) to deploy infrastructure for [Kubernetes](https://kubernetes.io/) on [KVM](https://wikipedia.org/wiki/KVM).

## Install the necessary components and dependencies

I am writing this guide 08.2019, versions of some components could not coincide with the current ones. All commands run as root.

###Operating system

For this project, I used CentOS operating system on hardware and virtual machines.

* [CentOS-7-x86_64-Minimal-1810](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso) for bare metal
* [CentOS-7-x86_64-GenericCloud-1901](https://cloud.centos.org/centos/7/images/) for virtual machines

### Check CPU virtualization 

First check if your CPU supports hardware virtualization

```
cat /proc/cpuinfo | egrep "(vmx|svm)"
```

If the command does not return anything, virtualization is not supported on the server or it is disabled in the BIOS settings. KVM itself can be put on such a server, but when we try to enter the hypervisor management command, we will get the error "WARNING KVM acceleration not available, using 'qemu'".

### Install libvirt and virsh

```
yum install -y qemu-kvm libvirt virt-install
```
* qemu-kvm - hypervisor; 
* libvirt - virtualization management library;
* virt-install - utility for managing virtual machines.

Allow autorun:

```
systemctl enable libvirtd
```

Launch KVM:

```
systemctl start libvirtd
```

### Network configuration

Install the package to work with bridge:

```
yum install -y bridge-utils
```

Сheck the real network interface with the configured IP address

```
ip a
```

Edit the settings of the real adapter:

```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

Need to get something like this:

```
TYPE=Ethernet
DEVICE=eth0
#IPADDR=192.168.1.100
#PREFIX=24
#GATEWAY=192.168.1.1
#DNS1=8.8.8.8
#DNS2=8.8.4.4
ONBOOT=yes
BOOTPROTO=none
BRIDGE=br0
```

Create an interface for the network bridge:

```
vi /etc/sysconfig/network-scripts/ifcfg-br0
```

Need to get something like this:

```
DEVICE=br0
TYPE=Bridge
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```
Restart the network:

```
systemctl restart network
```

Restart libvirtd:

```
systemctl restart libvirtd
```

### Install terraform

Ensure wget and unzip are installed

```
yum install -y wget unzip
```

Then download the terraform archive

```
wget https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip
unzip terraform_0.11.10_linux_amd64.zip
```

This will create a terraform binary file on your working directory. Move this file to the directory/usr/local/bin

```
mv terraform /usr/local/bin/
```

Confirm the version installed

```
terraform -v
```

### Install Terraform KVM provider

Terraform has a number of officially [supported providers](https://www.terraform.io/docs/providers/) available for use. Unfortunately, KVM is not in the list. I will use the [Terraform KVM provider](https://github.com/dmacvicar/terraform-provider-libvirt)

```
wget https://github.com/dmacvicar/terraform-provider-libvirt/releases/download/v0.5.1/terraform-provider-libvirt-0.5.1.CentOS_7.x86_64.tar.gz
tar xvf terraform-provider-libvirt-0.5.1.CentOS_7.x86_64.tar.gz
mv terraform-provider-libvirt ~/.terraform.d/plugins/
```

Check that libvirt daemon 1.2.14 or newer is running on the hypervisor

```
yum info libvirt
```

## Component versions

* OS - CentOS Linux release 7.6.1810 (Core)
* Libvirt - Version : 4.5.0
* Terraform - v0.11.10
* terraform-provider-libvirt - 0.5.1.CentOS_7.x86_64

## If all the necessary components and dependencies are installed, you can start writing the infrastructure as code!

Initialize a Terraform working directory

```
terraform init
```

Generate and show Terraform execution plan

```
terraform plan
```

Then build your Terraform infrastructure

```
terraform apply
```

Check your infrastructure use virsh

```
virsh list --all
``` 

You can destroy your Terraform infrastructure

```
terraform destroy
```