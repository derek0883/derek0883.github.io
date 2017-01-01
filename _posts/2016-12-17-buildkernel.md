---
layout: post 
title: "Build minimal kernel for QEMU"
categories: Virtualization 
comments: true
---

> There are many advantages to use QEMU to study kernel and device driver. e.g. hardware are emulated by QEMU, you can load QEMU with gdb,
then observe what happen when driver write a specific register of a "chip". We can change kernel source code, then build kernel,
and load with QEMU to debug it. I want keep the kernel image size as small as possible.

## Build minimal kernel for QEMU

Download kernel source code. I check out source code from github:

```bash
$ git clone https://github.com/torvalds/linux.git
$ cd linux 

# create a testing branch and check out this branch
$ git branch testing
$ git checkout testing
```

Now build a kernel using the default x86_64 configuration, this will use linux default configuration file arch/x86/configs/x86_64_defconfig
to create obj/x86_64/.config. 

```bash
$ make O=../obj/x86_64 x86_64_defconfig
```
../obj/x86_64 is output directory, it is not good to put this inside source code tree, e.g. O=obj/x86_64. 
as it will create a link in obj/x86_64, if obj/x86_64 stay inside kernel source code, this creates a recursive loop for grep or some other utilities.

```bash
$ readlink ../obj/x86_64/source
/home/ubuntu/OpenSource/kernel/linux
```
As I want build a kernel for guest os, so I'd like to merge kernel/configs/kvm_guest.config to .config. 

```bash
$ make O=../obj/x86_64 kvmconfig
```

scripts/kconfig/merge_config.sh will handle this, understand how this works is important, 
as we can create our own config file, then merge it. So I don't need do make menuconfig and search one by one.

```makefile
# scripts/kconfig/Makefile

%.config: $(obj)/conf
    $(if $(call configfiles),, $(error No configuration exists for this target on this architecture))
    $(Q)$(CONFIG_SHELL) $(srctree)/scripts/kconfig/merge_config.sh -m .config $(configfiles)
    +$(Q)yes "" | $(MAKE) -f $(srctree)/Makefile oldconfig
PHONY += kvmconfig
kvmconfig: kvm_guest.config
    @:
```

Now I also want to add some other config for debugging purpose, e.g. CONFIG_DEBUG_INFO, 
I don't want to search in menuconfig, and deal with dependency. 
So I add a new file: kernel/configs/testing.config

```
CONFIG_DEBUG_INFO=y 
CONFIG_IRQ_DOMAIN_DEBUG=y
CONFIG_DYNAMIC_DEBUG=y
CONFIG_BLK_DEV_RAM=y
CONFIG_ACPI_DEBUG=y
```

And change scripts/kconfig/Makefile to handle this.

```diff
diff --git a/scripts/kconfig/Makefile b/scripts/kconfig/Makefile
index 90a091b..d230927 100644
--- a/scripts/kconfig/Makefile
+++ b/scripts/kconfig/Makefile
@@ -122,6 +122,11 @@ PHONY += kvmconfig
 kvmconfig: kvm_guest.config
        @:
 
+PHONY += testingconfig
+testingconfig: testing.config
+       @:
+
+
 PHONY += xenconfig
 xenconfig: xen.config
        @:
```

Now merge my customized config.

```bash
$ make O=../obj/x86_64 testingconfig
```

Build kernel:

```bash
$ make O=../obj/x86_64 -j4
```

After it finished, copy ../obj/x86_64/arch/x86/boot/bzImage to QEMU directory. We also need a root filesystem to boot new kernel, 
you can [download busybox](https://www.busybox.net/){:target="_blank"} and compile by youself.
My goal is study kernel, so I download from binary x86_64-initramfs from [here](http://download.cirros-cloud.net/){:target="_blank"}. 

```bash
qemu-2.8.0-rc0 $ ./x86_64-softmmu/qemu-system-x86_64 \
	-kernel images/bzImage -initrd images/cirros-x86_64-initramfs \
	-nographic -append "console=ttyS0"
```

Now we can boot with kernel with qemu. both qemu and kernel were compile by myself, So I can change source code of either one, and load with gdb.
This makes much easier to study kernel code. for how to compile qemu, check my [previous post]({% post_url 2016-11-26-qemu-tracing1 %}){:target="_blank"}.

By default, during bootup cirros-x86_64-initramsf will try to connect to http://169.254.169.254, will retry 20 times.
I need wait few minutes for login prompt. I use following command re-pack initramfs.

```
$ mkdir initramfs
$ cd initramsfs 
$ sudo zcat ../cirros-x86_64-initramfs | sudo cpio -i
```

then open etc/cirros-init/config, delete ec2 from DATASOURCE_LIST.

```
# change 
DATASOURCE_LIST="nocloud configdrive ec2"
# to 
DATASOURCE_LIST="nocloud configdrive"
```

re-pack initramfs

```
$ sudo find . | sudo cpio -o -H newc | gzip > ../cirros-x86_64-initramfs
```

Now it will boot much faster.

At the end, checkin those changes to my own branch.

```bash
$ git add kernel/configs/testing.config
$ git commit -a -m "add my kernel configuration"
```

## Change password for cloud image
For kernel study purpose, I don't want spend too much time to build my own rootfile system,
cirros is small and fast, but it is build with busybox, sometime I want run some other command, e.g. gdb in guest system.
so I need a distribution linux as root filesystem. There are many options available on 
[Openstack](http://docs.openstack.org/image-guide/obtain-images.html){:target="_blank"}
I will use ubuntu, download xenial-server-cloudimg-amd64-disk1.img. 
By default it doesn't allow you to login with password, you need change password for cloud image.
follow [this link](https://access.redhat.com/discussions/664843){:target="_blank"} 
or [this link](http://docs.openstack.org/image-guide/modify-images.html){:target="_blank"}
to change password for cloud image.

```bash
$ sudo apt install guestfish libguestfs-tools

$ sudo guestfish --rw -a xenial-server-cloudimg-amd64-disk1.img
><fs> run
><fs> list-filesystems
><fs> mount /dev/sda1 /
><fs> vi /etc/shadow
```

On another terminal, you can replace password to other string, this is the password you need to login to guest.

```bash
$ openssl passwd -1 password
$1$qku9hhg8$KBrgkCQu8Hn8InuBX/9W1/
```

Now copy the encrypted output to /etc/shadow file. Change root and ubuntu

```
root:*:17148:0:99999:7:::
ubuntu:*:17148:0:99999:7:::
```

to

```
root:$1$qku9hhg8$KBrgkCQu8Hn8InuBX/9W1/:17148:0:99999:7:::
ubuntu:$1$qku9hhg8$KBrgkCQu8Hn8InuBX/9W1/:17148:0:99999:7:::
```

Now quit and boot your kernel with ubuntu root file system.

```bash
><fs> umount /
><fs> quit

qemu-2.8.0-rc0 $ ./x86_64-softmmu/qemu-system-x86_64 \
		-kernel bzImage -hda xenial-server-cloudimg-amd64-disk1.img -nographic \
		-append "console=ttyS0 root=/dev/sda1"
```

Then you can login with username: ubuntu, and password: password.

You can add the following to /etc/profile

```bash
if [ -e /usr/share/terminfo/x/xterm-256color ]; then
        export TERM='xterm-256color'
else
        export TERM='xterm-color'
fi
```

## Disable cloud-init
I don't need cloud-init, it just takes too long to boot up. 

```bash
$ sudo guestfish --rw -a xenial-server-cloudimg-amd64-disk1.img
><fs> run
><fs> list-filesystems
><fs> mount /dev/sda1 /
><fs> touch /etc/cloud/cloud-init.disabled
><fs> umount /
><fs> quit
```

Or you can boot with cloud-init=disabled on kernel command line.

```bash
qemu-2.8.0-rc0 $ ./x86_64-softmmu/qemu-system-x86_64 --enable-kvm \
		-kernel bzImage -hda xenial-server-cloudimg-amd64-disk1.img -nographic \
		-append "console=ttyS0 root=/dev/sda1 cloud-init=disabled"
```
