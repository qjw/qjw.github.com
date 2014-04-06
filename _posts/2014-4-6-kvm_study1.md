---
layout: post
title: KVM使用学习
category: other
---

##安装必要软件

	aptitude install gtkvncviewer qemu-system-x86 qemu-system-x86-64 qemu-utils

##基本运行

	qemu-system-i386 -cpu core2duo -m 512 -cdrom /media/f/debian-6.0.7-i386-netinst.iso -vnc :0 -name test

---

	qemu-system-i386 -cpu help
	x86           qemu64  QEMU Virtual CPU version 1.5.0                  
	x86           phenom  AMD Phenom(tm) 9550 Quad-Core Processor         
	x86         core2duo  Intel(R) Core(TM)2 Duo CPU     T7700  @ 2.40GHz 
	x86            kvm64  Common KVM processor                            
	x86           qemu32  QEMU Virtual CPU version 1.5.0                  
	x86            kvm32  Common 32-bit KVM processor                     
	x86          coreduo  Genuine Intel(R) CPU           T2600  @ 2.16GHz 
	x86              486                                                  
	x86          pentium                                                  
	x86         pentium2                                                  
	x86         pentium3

	-m megs         set virtual RAM size to megs MB [default=128]
	-cdrom file     use 'file' as IDE cdrom image (cdrom is ide1 master)
	-vnc display    start a VNC server on display
	-name string1[,process=string2]
		        set the name of the guest
		        string1 sets the window title and string2 the process name (on Linux)


以上代码分别指定了CPU类型，内存大小（M），光驱的盘片，以及vnc绑定的地址。

若不使用vnc，那么会自动弹出一个窗口来输出信息

:0 表示绑定当前的IP（通常总是可以用127.0.0.1连进去，端口是默认的5900 + 0**偏移**)

若指定unix路径，可以 **-vnc unix:/tmp/fuck**

##指定磁盘

	dd if=/dev/zero bs=1M count=128 of=sda
	qemu-img create -f raw -o size=128M sdb
	qemu-img create -f qcow2 -o size=1G sdc
	qemu-system-i386 -cpu core2duo -m 512 -cdrom pe.iso -hda sda -hdb sdb -hdd sdc -boot order=d

	qemu-system-i386 -cpu core2duo -m 512 -name fuck \
		-drive file=pe.iso,index=0,media=cdrom \
		-drive file=sda,index=1,media=disk,format=raw \
		-drive file=sdb,index=2,media=disk,format=raw \
		-drive file=sdc,index=3,media=disk,format=qcow2


---

	-hda/-hdb file  use 'file' as IDE hard disk 0/1 image
	-hdc/-hdd file  use 'file' as IDE hard disk 2/3 image
	-cdrom file     use 'file' as IDE cdrom image (cdrom is ide1 master)

	-boot [order=drives][,once=drives][,menu=on|off]
	      [,splash=sp_name][,splash-time=sp_time][,reboot-timeout=rb_time][,strict=on|off]
		        'drives': floppy (a), hard disk (c), CD-ROM (d), network (n)
		        'sp_name': the file's name that would be passed to bios as logo picture, if menu=on
		        'sp_time': the period that splash picture last if menu=on, unit is ms
		        'rb_timeout': the timeout before guest reboot when boot failed, unit is ms

##下载编译源码

	git clone git://git.kernel.org/pub/scm/virt/kvm/qemu-kvm.git
	aptitude install libglib2.0 libglib2.0-dev
	./configure --disable-werror --enable-debug --target-list=i386-softmmu
	make -j2



