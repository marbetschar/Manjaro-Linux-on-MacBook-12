First, install the linux-headers package for the 4.20.17-1-MANJARO kernel

Boot system with stock 4.20.17-1 kernel
**If you're not booted to this kernel then just replace '$(uname -r)' below with the full kernel name
Then download the kernel source tarball from https://www.kernel.org/

```
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.20.17.tar.xz
```

extract the drivers/bluetooth directory
cd to bluetooth directory
backup the hci_bcm.c file (optional)

```
cp  hci_bcm.c hci_bcm.c.orig
```

replace the Makefile with this one

```
obj-m 		+= hci_uart.o
hci_uart-y	:= hci_ldisc.o
hci_uart-m 	+= hci_serdev.o
hci_uart-m 	+= hci_h4.o
hci_uart-m	+= hci_bcsp.o
hci_uart-m	+= hci_ll.o
hci_uart-m	+= hci_ath.o
hci_uart-m	+= hci_h5.o
hci_uart-m	+= hci_intel.o
hci_uart-m	+= hci_bcm.o
hci_uart-m	+= hci_qca.o
hci_uart-m	+= hci_ag6xx.o
hci_uart-m	+= hci_mrvl.o
hci_uart-objs	:= $(hci_uart-y)

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

put the contents of this patch in a file called patchfile (or whatever you want to call it)
https://github.com/christophgysin/linux/commit/b8c43916bafe6dc8de1aed5ae1cbc76cf7bad374.patch
now patch hci_bcm.c

(to create a path: `diff -u a/drivers/bluetooth/hci_bcm.c b/drivers/bluetooth/hci_bcm.c > linux-4.19.32_drivers-bluetooth-hci_bcm.patch`)

```
patch < patchfile
```

compile hci_uart.ko module

```
[root@mac bluetooth]# make
make -C /lib/modules/4.20.16-200.fc29.x86_64/build M=/root/4.20.bluetooth modules
make[1]: Entering directory '/usr/src/kernels/4.20.16-200.fc29.x86_64'
  CC [M]  /root/4.20.bluetooth/hci_ldisc.o
  CC [M]  /root/4.20.bluetooth/hci_serdev.o
  CC [M]  /root/4.20.bluetooth/hci_h4.o
  CC [M]  /root/4.20.bluetooth/hci_bcsp.o
  CC [M]  /root/4.20.bluetooth/hci_ll.o
  CC [M]  /root/4.20.bluetooth/hci_ath.o
  CC [M]  /root/4.20.bluetooth/hci_h5.o
  CC [M]  /root/4.20.bluetooth/hci_intel.o
  CC [M]  /root/4.20.bluetooth/hci_bcm.o
  CC [M]  /root/4.20.bluetooth/hci_qca.o
  CC [M]  /root/4.20.bluetooth/hci_ag6xx.o
  CC [M]  /root/4.20.bluetooth/hci_mrvl.o
  LD [M]  /root/4.20.bluetooth/hci_uart.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /root/4.20.bluetooth/hci_uart.mod.o
  LD [M]  /root/4.20.bluetooth/hci_uart.ko
make[1]: Leaving directory '/usr/src/kernels/4.20.16-200.fc29.x86_64'
```

compress module (optional)

```
xz hci_uart.ko
```

install module

```
cp hci_uart.ko.xz /lib/modules/$(uname -r)/kernel/drivers/bluetooth/
```

reboot...


