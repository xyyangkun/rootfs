spi flash使用jffs2格式的镜像，制作jffs2镜像时，需要用到spi flash的块大小。这些信息会在uboot启动时会打印出来。建议使用时先直接运行mkfs.jffs2工具，根据打印信息填写相关参数。下面以块大小为64KB为例：
	./tool/mkfs.jffs2 -d ./rootfs_uclibc -l -e 0x10000 -o ./rootfs_uclibc_64k.jffs2 -p 1800000
(64k的块)
sudo ./tool/mkfs.jffs2 -d ./rootfs_gclibc/ -l -e 0x10000 -p 1800000 -o ./rootfs_gclibc_64k.jffs2


sudo ./tool/mkfs.jffs2 -d ./rootfs_mini/ -l -e 0x10000 -p 2700000 -o ./rootfs.jffs2


16 =0x10
256=0x100
4K =0x1000
64K=0x10000
1M =0x100000
16M=0x1000000
烧写映像文件到SPI Flash
以16M SPI Flash为例。
1）地址空间说明
    |      512K     |      4M       |      24M      |
    |---------------|---------------|---------------|
    |     boot      |     kernel    |     rootfs    |

    以下的操作均基于图示的地址空间分配，您也可以根据实际情况进行调整。
2）烧写u-boot
    sf probe 0
    mw.b 82000000 ff 80000
    tftp 0x82000000 u-boot.bin
    sf probe 0
    sf erase 0 80000
    sf write 82000000 0 80000	
    reset    
3）烧写内核
    mw.b 82000000 ff 400000
    tftp 82000000 uImage
    sf probe 0
    sf erase 80000 400000
    sf write 82000000 80000 400000
4)烧写文件系统
    mw.b 82000000 ff 2000000
    tftp 0x82000000 rootfs.jffs2
    sf erase 480000 1800000
    sf write 82000000 480000 180000
5）设置启动参数
    setenv bootargs 'mem=64M console=ttyAMA0,115200 root=/dev/mtdblock2 rootfstype=jffs2 mtdparts=hi_sfc:512K(boot),4M(kernel),24M(rootfs)'
    setenv bootcmd 'sf probe 0;sf read 0x82000000 0x80000 0x400000;bootm 0x82000000'
    sa
    

可用：
setenv bootargs 'mem=64M console=ttyAMA0,115200 root=/dev/mtdblock2 rootfstype=jffs2 mtdparts=hi_sfc:512K(boot),4M(kernel),27M(rootfs)'



setenv bootargs 'mem=64M console=ttyAMA0,115200 root=/dev/sda1 rootfstype=ext4 '

setenv bootargs 'mem=108M console=ttyAMA0,115200 root=/dev/mtdblock2 ro rootfstype=cramfs  mtdparts=hi_sfc:512K(boot),4M(kernel),6656K(rootfs)'

setenv bootargs 'mem=64M console=ttyAMA0,115200 root=/dev/sda1 rw rootfstype=ext4 init=/usr/lib/initramfs-tools/bin/busybox'



    setenv bootargs 'mem=64M console=ttyAMA0,115200 root=/dev/mtdblock2 rootfstype=jffs2 mtdparts=hi_sfc:512K(boot),4M(kernel),4M(rootfs)'
    
    
