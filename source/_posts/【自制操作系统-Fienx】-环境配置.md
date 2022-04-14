---
title: 【自制操作系统-Fienx】-环境配置
date: 2022-04-11 21:46:25
tags:
  - Fienx
  - 操作系统
categories:
  - 操作系统
  - Fienx
---

## dd

+ 下载命令行工具：[dd](http://www.chrysocome.net/dd)，并解压。
+ 将dd.exe的父目录添加到环境变量PATH。

## NASM

+ 下载[NASM](https://www.nasm.us/)，以管理员模式为所有用户安装。
+ 将nasm.exe的父目录添加到环境变量PATH。

## Bochs

+ 下载[Bochs](https://sourceforge.net/projects/bochs/files/bochs/)，一路以默认选项Next，安装完成。
+ 将bochs.exe的父目录设为环境变量BOCHS_ROOT。
+ 将BOCHS_ROOT添加到环境变量PATH下。

## “最小”的操作系统

+ 编写bootsect.asm

  ```x86asm
          org     07c00h
          mov     ax, cs
          mov     ds, ax
          mov     es, ax
          call    DispStr
          jmp     $
  DispStr:
          mov     ax, BootMessage
          mov     bp, ax
          mov     cx, 16
          mov     ax, 01301h
          mov     bx, 000ch
          mov     dl, 0
          int     10h
          ret
          ; 定义BootMessage
  BootMessage:
          db      "Hello, OS world!"
          ; 填充0到510字节处
          times   510-($-$$)  db  0
          ; 0xaa55是结束标志，占2字节
          dw      0xaa55
  ```

+ 创建一个软盘映像：在管理员模式下运行Bochs文件夹下的bximage.exe。

  1. Create new floppy or hard disk image
  2. fd
  3. 1.44M
  4. fienx.img

+ 使用NASM编译[bootsect.asm](void)。

  ```bash
  $ nasm bootsect.asm -o bootsect.bin
  ```

+ 使用dd将[bootsect.bin](void)写入到[fienx.img](void)。

  ```bash
  $ dd if=bootsect.bin of=fienx.img bs=512 count=1 conv=notrunc
  ```

+ 编写配置文件[bochsrc.bxrc](void)。

  ```yaml
  ###############################################################
  # bochsrc.txt file for DLX Linux disk image.
  ###############################################################
  
  # how much memory the emulated machine will have
  megs: 32
  
  # filename of ROM images
  romimage: file=$BOCHS_ROOT/BIOS-bochs-latest
  vgaromimage: file=$BOCHS_ROOT/VGABIOS-lgpl-latest
  
  # what disk images will be used 
  floppya: 1_44=fienx.img, status=inserted
  
  # choose the boot disk.
  boot: floppy
  
  # where do we send log messages?
  log: bochsout.txt
  
  # disable the mouse, since DLX is text only
  mouse: enabled=0
  
  # set up IPS value and clock sync
  cpu: ips=15000000
  clock: sync=both
  
  # enable key mapping, using US layout as default.
  #
  # NOTE: In Bochs 1.4, keyboard mapping is only 100% implemented on X windows.
  # However, the key mapping tables are used in the paste function, so 
  # in the DLX Linux example I'm enabling keyboard_mapping so that paste 
  # will work.  Cut&Paste is currently implemented on win32 and X windows only.
  
  keyboard: keymap=$BOCHS_ROOT/keymaps/x11-pc-us.map
  #keyboard: keymap=../keymaps/x11-pc-fr.map
  #keyboard: keymap=../keymaps/x11-pc-de.map
  #keyboard: keymap=../keymaps/x11-pc-es.map
  ```
  
+ 使用bochs运行配置文件[bochsrc.bxrc](void)。

  ```bash
  $ bochs -f bochsrc.bxrc
  ```


> 参考资料：
>
> - 《Orange's：一个操作系统的实现》
>

