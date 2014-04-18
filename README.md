# Ganeti cloud-init

Provision Ganeti VMs with [cloud images](http://cloud-images.ubuntu.com)


## [Ganeti](http://code.google.com/p/ganeti/) is a cluster virtual server managment tool that leverages the KVM and XEN hypervisors.

[Ganeti cloud-init](https://github.com/pdxcat/ganeti_cloud-init) is a ganeti [os definition](http://docs.ganeti.org/ganeti/2.11/man/ganeti-os-interface.html) that creates ganeti instance from cloud images. Cloud images are Amazon/openstack-ready disk images containing pre-installed operating systems. They are produced regularly by all major linux distribution vendors.

The advantages to installing virtual machines from cloud images are:

* Speed: This is much faster than using PXE boot or cdrom boot to load virtual machines.
* Consistency: These images will be identical to amazon AMIs or machines running on an openstack cluster.
* Lower effort: The OSUOSL Ganeti-instance-image project can produce a similar effect, but requires the administrator to provide their own dump/image files. These images can be pulled from the internet.


## How To Use

### Requirements

Currently this provisioner uses the cloud-localds that is provided by Ubuntu version 0.27 of the cloud-utils package. This utility requires genisoimage to work.

If your system doesn't have cloud-localds, you can download it [here](https://gist.github.com/cpswan/6221258)

This has only been tested using a KVM hypervisor, YMMV with other hypervisors.

### Configuration and Files

#### variants.list
This file is a newline delimited list of the cloud images that you support. 

#### environment.sh
Site specific information should be set here.
* *cloud\_images* The directory where your cloud image files will live.
* *root_ssh_authorized_keys* The ssh authorized_keys that will allow sshing into the vms as root

#### cloud\_images 
Should contain a raw type (not qcow) cloud image for each os variant you support. Files should be named:
```
${OS_VARIANT}.img.raw
```
Information on how to convert a cloud image to a raw disk image can be found [in ubuntu's docs](https://help.ubuntu.com/community/UEC/Images#Ubuntu_Cloud_Guest_images_on_12.04_LTS_.28Precise.29_and_beyond_using_NoCloud).

### Creating a VM

This provisioner works using [NoCloud](http://cloudinit.readthedocs.org/en/latest/topics/datasources.html#no-cloud) so each VM you create should at least initially have two disks, one for the root file-system and one for the NoCloud ISO image. Since Ganeti seems to not be able to allocate disks smaller than one 1GB and the NoCloud ISO is under 1MB it is recommended that you remove the second disk after provisioning the VM if disk space is a concern for you.

At least with the Ubuntu cloud images, the system will resize the root partition to fill the whole disk that you give the VM. So allocating a disk 0 of size 10GB will give you a 10GB root file-system.

Your disk 0 should be at least the same size as the cloud image for the variant you are loading. In the future this will be enforced by the provisioner. If your disk 0 is smaller than your cloud image your VM might fail to boot, or worse: fail unexpectedly during use.

#### 'gnt-instance add' Example

```
gnt-instance add --disk-template plain --disk 0:size=15G --disk 1:size=1G --hypervisor-parameters kvm:boot_order=disk --os-type cloud-init+ubuntu-trusty example.com
```
