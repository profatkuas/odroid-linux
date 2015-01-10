# Real-time kernel for ODROID U3
- applied edit 4 from http://forum.odroid.com/viewtopic.php?f=79&t=7453, see below
- changed sound/soc/samsung/dma.c to allow for smaller buffersizes
- configure with make odroidu_defconfig
- change configuation with make menuconfig
-- disable Ubuntu supplied Third-Party device drivers -> Aufs (unselect) (see http://forum.odroid.com/viewtopic.php?f=79&t=7453)
- make with: make -j4 zImage modules
- install with: sudo make modules_install && sudo cp arch/arm/boot/zImage /media/boot/

# Setting up JACK
- xterm sudo synaptic
-- add qjackctl (fetches also JACK and other dependencies)

# References
Instructions from http://forum.odroid.com/viewtopic.php?f=79&t=7453, which did not work for me :(

1. Get the latest HK kernel source for odroidu:
CODE: SELECT ALL
git clone https://github.com/hardkernel/linux.git -b odroid-3.8.y --depth=1 odroidu-3.8.y

2. Get the RT patch for 3.8.y from kernel.org and apply it:
CODE: SELECT ALL
cd odroidu-3.8.y
wget https://www.kernel.org/pub/linux/kernel/projects/rt/3.8/patch-3.8.13.14-rt31.patch.xz
xz -d patch-3.8.13.14-rt31.patch.xz
patch -p1 < {path to .patch file}

3. Resolve the rejects.
1st failure drivers/md/raid5.c.rej
do a spin_lock_init(&conf->percpu->lock) instead before the ifdef at line 5158

2nd failure seen in include/linux/irqdesc.h.rej
Trivial merge: add u64 random_ip; after irqs_unhandled;

3rd failure seen in kernel/irq/manage.c
Trivial merge: add the rej file from line 163 etc.

4. Add the UART OOPs patch for samsung
Basically in function s3c24xx_serial_rx_chars() in drivers/tty/serial/samsung.c:
+ spin_unlock(&port->lock);
tty_flip_buffer_push(tty);
+ spin_lock(&port->lock);

5. Compile kernel
CODE: SELECT ALL
make odroidu_defconfig
make menuconfig

Kernel Feature -> Preemption Model -> Fully Preemptible kernel
Also disable Aufs for now as it gives a compilation issue.
Ubuntu supplied Third-Party device drivers -> Aufs (unselect)
CODE: SELECT ALL
make -j <whatever> zImage modules


6. Update SD/eMMC card with kernel and modules.
VFAT 1st partition -> save zImage as zImage.org and copy the compiled zImage from arch/arm/boot/zImage in its place.
make modules_install (to install the modules)

7. Reboot and voila!
CODE: SELECT ALL
root@odroid:~# uname -a
Linux odroid 3.8.13.28-rt31 #3 SMP PREEMPT RT Sat Dec 13 10:33:43 PST 2014 armv7l armv7l armv7l GNU/Linux
