# 查询命令

## bdinfo

- 查看板子信息


## printenv

- 输出环境变量信息

> 后面加变量名打印对应变量

## version

- 查看 uboot 的版本号


# 环境变量操作命令


## setenv

setenv [变量] [值]

- 设置值

多个*值*之间用**空格**隔开，所以需要使用*单引号* '' 将其括起来

删除环境变量可以通过 setenv 给 [变量] 赋空值

## saveenv

- 保存值

# 内存操作命令

内存操作命令就是用于直接对 DRAM 进行读写操作

## md

`md[.b, .w, .l] address [# of objects]`

- 显示内存值

命令中的 [.b .w .l] 对应分别以 1 个字节、 2 个字节、 4 个字节来显示内存值

address 就是要查看的内存起始地址

[# of objects]表示要查看的数据长度，长度单位是*个*

**uboot 命令中的数字都是十六进制的**

例如 查看 0XC0100000 开始的 20 个字节的内存值，显示格式为.b ：

> md.b C0100000 14

## nm

`nm [.b, .w, .l] address`

- 用于修改指定地址的内存值

## mm

- 修改指定地址内存值，修改内存值地址会**自增**

输入 q 结束修改

## mw

`mw [.b, .w, .l] address value [count]`

- 使用一个指定的数据填充一段内存

*value* 为要填充的数据，*count* 是填充的长度

例如 使用*.l* 格式将以 *0XC0100000* 为起始地址的 *0x10* 个内存块(0x10 * 4=64 字节)填充为 *0X0A0A0A0A*

> mw.l C0100000 0A0A0A0A 10

## cp

`cp [.b, .w, .l] source target count`

- 将 DRAM 中的数据从一段内存拷贝到另一段内存中，或者把 NorFlash 中的数据拷贝到 DRAM

*source* 为源地址， *target* 为目的地址， *count* 为拷贝的长度

## cmp

`cmp [.b, .w, .l] addr1 addr2 count`

- 比较两段内存的数据是否相等

*addr1* 为第一段内存首地址， *addr2* 为第二段内存首地址， *count* 为要比较的长度

# 网络操作命令

## ping

## dhcp

- dhcp 用于从路由器获取 IP 地址
- 前提是开发板得连接到路由器上的

## nfs

- 通过 nfs 可以在计算机之间通过网络来分享资源

例如 将Ubuntu主机的 uImage 下载到开发板 DRAM 的 0XC2000000 地址处

>nfs C2000000 192.168.1.249:/home/zuozhongkai/linux/nfs/uImage

## tftp

- 通过网络下载东西到 DRAM 中，只是 tftp 命令使用的是 TFTP 协议

例如：tftp C2000000 uImage

# EMMC 和 SD 卡操作命令

一般认为 EMMC 和 SD 卡是同一个东西，所以没有特殊说明，所以统一使用 MMC 来代指 EMMC 和 SD 卡

> *mmc* 是一系列的命令，其后可以跟不同的参数

## mmc info

- 输出 MMC 设备信息

## mmc rescan

- 扫描当前开发板上所有的 MMC 设备

## mmc list

- 查看当前开发板一共有几个 MMC 设备

## mmc dev

`mmc dev [dev] [part]`

- 用于切换当前 MMC 设备

> [dev]用来设置要切换的 MMC 设备号， [part]是分区号， 如果不写分区号的话默认为分区 0
> 例如 切换到 SD 卡：
> mmc dev 0

## mmc part

- 查看 MMC 设备分区

输入如下命令：
> mmc dev 1 //切换到 EMMC
> mmc part //查看 EMMC 分区

## mmc read

`mmc read addr blk# cnt`

- 读取 mmc 设备的数据

*addr* 是数据读取到 DRAM 中的地址

*blk* 是要读取的块起始地址(十六进制)，一个块是 512字节，这里的块和扇区是一个意思，在 MMC 设备中通常说扇区， 

*cnt* 是要读取的块数量(十六进制)

# EXT 格式文件系统操作命令

uboot 有 ext2 和 ext4 这两种格式的文件系统的操作命令,STM32MP1 的系统镜像都是 ext4 格式的

## ext4ls

- 查询 EXT4 格式设备的目录和文件信息

`ext4ls <interface> [<dev[:part]>] [directory]`

interface	要查询的接口，比如 mmc
dev 是要查询的设备号
part		要查询的分区
directory	要查询的目录

比如查询 EMMC 分区 2 中的所有的目录和文件

> ext4ls mmc 1:2

## ext4load

- 将指定的文件读取到 DRAM 中

`fatload <interface> [<dev[:part]> [<addr> [<filename> [bytes [pos]]]]]`

- interface	接口，比如 mmc， 
- dev 		设备号
- part		分区
- addr		保存在 DRAM 中的起始地址
- filename	要读取的文件名字
- bytes		表示读取多少字节的数据，如果 bytes 为 0 或者省略的话表示读取整个文件
- pos		要读的文件相对于文件首地址的偏移，如果为 0 或者省略的话表示从文件首地址开始读取。

> 例如 EMMC 分区 2 中的 uImage 文件读取到 DRAM 中的 0XC2000000 地址处
> ext4load mmc 1:2 C2000000 uImage

## ext4write

- 将 DRAM 中的数据写入到 MMC 设备中

`ext4write <interface> <dev[:part]> <addr> <absolute filename path> [sizebytes] [file offset]`

- addr 是要写入的数据在 DRAM中的起始地址
- absolute filename path 是写入的数据文件名字，注意是要带有绝对路径，以‘/’开始
- sizebytes 表示要写入多少字节的数据
- file offset 为文件偏移

# BOOT 操作命令

## bootm

- 启动 uImage 镜像文件

`bootm [addr [arg ...]]`

> addr    是 Linux 镜像文件在 DRAM 中的位置
> arg ... 表示其他可选的参数
>     比如要指定 initrd 的话，第二个参数就是 initrd 在 DRAM 中的地址。
>     如果 Linux 内核使用设备树的话还需要第三个参数，用来指定设备树在 DRAM 中的地址，如果不需要 initrd 的话第二个参数就用‘-’来代替。


## bootz

- 启动 zImage 镜像文件

`bootz [addr [initrd[:size]] [fdt]]`

addr 	是 Linux 镜像文件在 DRAM 中的位置
initrd 	是 initrd 文件在DRAM 中的地址，如果不使用 initrd 的话使用‘-’代替即可
fdt 	是设备树文件在 DRAM 中的地址

> 使用方法和 bootm 一模一样，只是所引导的 Linux 镜像格式不同


# UMS 命令

- 将开发板虚拟成一个 U 盘，可以选择使用哪个 Flash 作为这个 U 盘的存储器

`ums <USB_controller> [<devtype>] <dev[:part]>`

- USB_controller 是 usb 接口索引
	有的开发板有多个 USB SLAVE 接口，具体要使用哪个就可以通过 USB_controller 参数指定。 正点原子 STM32MP157 开发板的只有一个 USB_OTG口可以作为 USB SLAVE， 对应的索引为 0。 

- Devtype 		是要挂载的设备，默认为 mmc
- dev[:part]	是要挂载的 Flash 设备
- part			是要挂载的分区。

> ums 0 mmc 1
> 终端下运行,按住 CTRL+C 键就能结束这个挂载

# 其他常用命令

## reset

- 复位

## go

- 命跳到指定的地址处执行应用

`go addr [arg ...]`

> addr 是应用在 DRAM 中的首地址

## run

- 运行环境变量中定义的命令

通过自定义环境变量来实现不同的启动方式

例如：

```sh
setenv mybootemmc 'ext4load mmc 1:2 c2000000 uImage;ext4load mmc 1:2 c4000000 stm32mp157d-atk.dtb;bootm c2000000 - c4000000'
setenv mybootnet 'tftp c2000000 uImage;tftp c4000000 stm32mp157d-atk.dtb;bootm c2000000 - c4000000'
saveenv
# 创建环境变量成功以后就可以使用 run 命令来运行 mybootemmc、 mybootnet 或 mybootnand 来实现不同的启动：
# run mybootemmc
# 或
# run mybootnet
```

## mtest

- 一个简单的内存读写测试命令，可以用来测试开发板上的 DDR

`mtest [start [end [pattern [iterations]]]]`

- start 是要测试的 DRAM 开始地址
- end 是结束地址

> 比如测试: 0XC0000000~0XC0001000 这段内存，输入 `mtest C0000000 C0001000`
> 结束测试就按下键盘上的“Ctrl+C”

## MII

- 主要用于读取网络 PHY 芯片寄存器
- 在 uboot 中调试网络 PHY 芯片的时候非常有用

> STM32MP157 需要每次使用 MII 或者 MDIO 命令来操作 PHY 芯片的时候，先使能 ETHMAC 时钟。将 0X50000218 这个寄存器的 bit8~10 置 1 即可，才能使用 mii 命令

`mii info addr`

- 显示 MII PHY 信息

addr 就是 PHY 芯片地址


## bmp

`bmp info <imageAddr>`

- 显示BMP图片信息

> imageAddr 就是 BMP 图片在 RAM 中的起始地址

`bmp display c0000000 0 0`

0 0 是显示的位置坐标

- 显示BMP图片

# bootargs环境变量

在 Linux 内核源码里面有相应的文档讲解如何设置，文档为 *Documentation/filesystems/nfs/nfsroot.txt*

格式： 

```
root=/dev/nfs nfsroot=[<server-ip>:]<root-dir>[,<nfs-options>] ip=<client-ip>:<server-ip>:<gwip>:<netmask>:<hostname>:<device>:<autoconf>:<dns0-ip>:<dns1-ip>
```

- <server-ip>：服务器 IP 地址，也就是存放根文件系统主机的 IP 地址，那就是 Ubuntu 的 IP 地址，比如我的 Ubuntu 主机 IP 地址为 192.168.1.249。
- <root-dir>： 根文件系统的存放路径，比如我的就是/home/zuozhongkai/linux/nfs/rootfs。
- <nfs-options>： NFS 的其他可选选项，一般不设置。
- <client-ip>： 客户端 IP 地址，也就是我们开发板的 IP 地址， Linux 内核启动以后就会使用此 IP 地址来配置开发板。此地址一定要和 Ubuntu 主机在同一个网段内，并且没有被其他的设备使用，在 Ubuntu 中使用 ping 命令 ping 一下就知道要设置的 IP 地址有没有被使用，如果不能 ping 通就说明没有被使用，那么就可以设置为开发板的 IP 地址，比如我就可以设置为 192.168.1.250。
- <server-ip>： 服务器 IP 地址，前面已经说了。
- <gw-ip>： 网关地址，我的就是 192.168.1.1。
- <netmask>：子网掩码，我的就是 255.255.255.0。
- <hostname>：客户机的名字，一般不设置，此值可以空着。
- <device>： 设备名，也就是网卡名，一般是 eth0， eth1….，正点原子 STM32MP157 开发板只有一个网口，名字为 eth0。
- <autoconf>： 自动配置，一般不使用，所以设置为 off。
- <dns0-ip>： DNS0 服务器 IP 地址，不使用。
- <dns1-ip>： DNS1 服务器 IP 地址，不使用。

根据上面的格式 bootargs 环境变量的 root 值如下：

```sh
root=/dev/nfs nfsroot=192.168.1.249:/home/zuozhongkai/linux/nfs/rootfs,proto=tcp rw ip=192.
168.1.250:192.168.1.249:192.168.1.1:255.255.255.0::eth0:off
```

- “proto=tcp”表示使用 TCP 协议
- “rw”表示 nfs 挂载的根文件系统为可读可写

# 手册勘误



287页倒数第3行：
tftp c2000000 stm32mp157d-atk.dtb >> tftp c4000000 stm32mp157d-atk.dtb


295页第3行：
mii info 0x4 >> mii info 0x0