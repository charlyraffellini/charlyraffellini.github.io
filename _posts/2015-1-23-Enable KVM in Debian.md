---
layout: post
title: Enable KVM in Debian and setup Android Studio.
---

### For Ubuntu go [here](https://software.intel.com/en-us/blogs/2012/03/12/how-to-start-intel-hardware-assisted-virtualization-hypervisor-on-linux-to-speed-up-intel-android-x86-emulator).

### Requirements
- To see if your processor supports hardware virtualization:
    - Run `egrep -c '(vmx|svm)' /proc/cpuinfo`.
    - If 0 your CPU does not support hardware virtualization.


- Then you have to check for harware virtualization support ([reference](https://nsrc.org/workshops/2014/sanog23-virtualization/raw-attachment/wiki/Agenda/ex-debian-kvm-libvirt.htm#check-for-hardware-virtualization-support)):
    - Run `egrep '(vmx|svm)' /proc/cpuinfo`, It shall return more than one line containing "vmx" or "svm".
    - Then run `ls -l /dev/kvm`, It shall return a line like `crw-rw---T+ 1 root kvm 10, 232 Jan 23 00:07 /dev/kvm`.


### KVM installation ([reference](https://wiki.debian.org/KVM))

- Run `sudo apt-get install qemu-kvm libvirt-bin virtinst kvm virt-viewer`
- Add your user to _kvm_ and _libvirt_ groups:
    - `sudo adduser <youruser> kvm`
    - `sudo adduser <youruser> libvirt`


### Verify installation
- Run the following: `sudo virsh -c qemu:///system list`. If the output is
		```
        Id Name                 State
        ----------------------------------
        ```
    Then is OK.


### Setup virtualization acceleration on Android Studio
- Go to Run > Edit Configurations...
- In Emulator tab check Additional command line options and copy `-qemu -m 2047 -enable-kvm`.
