# 0. TODO

- [ ] The shopping list
- [ ] Stuff you'll need to buy
- [x] fdisk instructions for Linux

# 1. What you'll need

## 1.1 Hardware shopping list:

In theory, you can use any combination of RPi models. I personally recommend getting at least 3 Pi's. Using any less makes it kind of boring, and people won't be nearly as impressed :smirk:

RPi 2 B's are recommended. I happened to get 4 RPi 2 B's, and I threw in an old RPi B+ model as well, mainly because I was curious about the performance on older models.

| Item          | Amount        | Price |
| ------------- |--------------:| -----:|
| RPi Model B+  | 1x            | $1600 |
| Rpi 2 Model B | 4x            |   $12 |

## 1.2 Tools and OS

- An SD Card writer
- A Linux distribution to format your SD card to ext4 and write the Arch Linux image
- If you're working on Windows, make sure you have PuTTY and know how to use it

# 2. Preparing the RPi's

## 2.1  Installing the OS

Most of the information I'll provide here is available over on the [official Arch ARM site](http://archlinuxarm.org/).

First, make sure you download the correct images from the Arch site.

* For the original **RPi images**, have a look at the overview over [here](http://archlinuxarm.org/platforms/armv6/raspberry-pi), or simply download the image [here](http://archlinuxarm.org/os/ArchLinuxARM-rpi-latest.tar.gz).

* For the newer **RPi 2 model**, , have a look at the overview over [here](http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2), or simply download the image [here](http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz).

Now, insert the SD card you want to use to boot your little Pi.

**NOTE!** If you're using a Linux Virtual Machine on VMWare your Mac to format the SD card, please note that VMWare Fusion does NOT support mounting the SD card to the VM, as it is not recognized as a USB device. If you want to work on a Virtual Machine with your Mac card reader, you'll have to get yourself a USB SD card reader.

**NOTE 2!** The instructions below are pretty much a direct copy of the official Arch guide, so make sure to reference that one when in doubt.

First things first: we'll need to find out the device name of the SD card. On most Linux systems, `sudo fisk -l` should give you the required info.

Here's my output as an example. You can see that my 16GB SD card was assigned /dev/sdb.

```
root@localhost ~]# fdisk -l

Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0008ff90

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048      616447      307200   83  Linux
/dev/sda2          616448     4810751     2097152   82  Linux swap / Solaris
/dev/sda3         4810752    41943039    18566144   83  Linux

Disk /dev/sdb: 15.8 GB, 15819866112 bytes, 30898176 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xf005aa26

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048      206847      102400    c  W95 FAT32 (LBA)
/dev/sdb2          206848    30898175    15345664   83  Linux
```

Now, start up fdisk targeting the correct device name, for example: `fdisk /dev/sdb`

**BEWARE!** On most Linux distributions and OS X, using fdisk requires you to execute them as root, and not just sudo the commands. Please be careful while taking these steps.

This will drop you into the interactive fdisk prompt. The goal is to clean the old partitions on the card and create a new ones.

1. First, clean the old partitions by creating a new partition type. Type `o`.

  ```
  Command (m for help): o
  Building a new DOS disklabel with disk identifier 0xd44377c5.
  ```

  Type `p` to see that the partition table is, indeed, empty.

  ```
  Command (m for help): p

  Disk /dev/sdb: 15.8 GB, 15819866112 bytes, 30898176 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disk label type: dos
  Disk identifier: 0xd44377c5

     Device Boot      Start         End      Blocks   Id  System

  ```

2. Now we're going to create a new boot partition. To do this, type `n`, and then type `p` to designate it as a primary partition. Type `1` to assign it as the very first partition, and then press `Enter` to accept the default first sector. Next, type in `+100M` as the final sector.

  ```
  Command (m for help): n
  Partition type:
     p   primary (0 primary, 0 extended, 4 free)
     e   extended
  Select (default p): p
  Partition number (1-4, default 1): 1
  First sector (2048-30898175, default 2048):
  Using default value 2048
  Last sector, +sectors or +size{K,M,G} (2048-30898175, default 30898175): +100M
  Partition 1 of type Linux and of size 100 MiB is set
  ```

  If you're curious you can once again type `p` to see that we indeed reserved a 100M boot partition.

  ```
  Command (m for help): p

  Disk /dev/sdb: 15.8 GB, 15819866112 bytes, 30898176 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disk label type: dos
  Disk identifier: 0xd44377c5

     Device Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048      206847      102400   83  Linux
  ```

3. We'll need to change the type of the above partition, so type `t`, followed by `c` to change the type to **W95 FAT32 (LBA)**.

  ```
  Command (m for help): t
  Selected partition 1
  Hex code (type L to list all codes): c

  WARNING: If you have created or modified any DOS 6.xpartitions, please see the fdisk manual page for additionalinformation.

  Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'
  ```

4. So let's create that second partition. You know the drill. Type `n`, then `p`. Make sure to make it the second partition by typing `2`. Then press `Enter` twice to accept both the default start and end sector (this will make this partition take up the space that's left on the card).

  ```
  Command (m for help): n
  Partition type:
     p   primary (1 primary, 0 extended, 3 free)
     e   extended
  Select (default p): p
  Partition number (2-4, default 2): 2
  First sector (206848-30898175, default 206848):
  Using default value 206848
  Last sector, +sectors or +size{K,M,G} (206848-30898175, default 30898175):
  Using default value 30898175
  Partition 2 of type Linux and of size 14.7 GiB is set
  ```

5. There we go. Now time to write the changes. Type `w`.

6. Right, now that we're out of the fdisk interactive prompt, let's mount the FAT boot partition.

  ```
  [root@localhost Downloads]# mkfs.vfat /dev/sdX1       # Fill in sdX1 according to your own setup
  [root@localhost Downloads]# mkdir boot
  [root@localhost Downloads]# mount /dev/sdX1 boot
  ```

  Then do the same for the Linux partition:

  ```
  [root@localhost Downloads]# mkfs.ext4 /dev/sdX2
  [root@localhost Downloads]# mkdir root
  [root@localhost Downloads]# mount /dev/sdX2 root
  ```

7. Next, we'll untar the .tar.gz containing the Arxh ARM image into the root folder where we mounted the root partition.

  ```
  [root@localhost Downloads]# bsdtar -xpf ArchLinuxARM-rpi-latest.tar.gz -C root
  ```

  Sync it up:
  ```
  [root@localhost Downloads]# sync
  ```

8. Now for the final task, let's move the files from root/boot over to boot.

  ```
  [root@localhost Downloads]# sudo mv root/boot/* boot
  ```

  That's it! Unmount it:
  ```
  [root@localhost Downloads]# umount boot root
  ```

## 2.2 Configuring the OS

**NOTE!** Please make sure you only use nmap at home. Using it at work might make some of your admins quite mad... :wink:

Now that you've hooked up your Raspberry Pi to the power and network, let's SSH into it. If you're a cheap bum like me, you might only have a monitor with an DisplayPort instead of an HDMI port. Luckily, we can easily find out the ip address of your Pi using nmap.

In my example, my local ip address is 192.168.1.225. So in order to scan my entire subnet, I can use nmap like so:

```
[root@localhost ~]# sudo nmap -sn 192.168.1.0/24
```

You'll get some info along these lines:

```
Starting Nmap 7.00 ( https://nmap.org ) at 2015-12-17 17:14 CET

  ...

MAC Address: B8:27:EB:D7:FD:ED (Raspberry Pi Foundation)
Nmap scan report for 192.168.1.241
Host is up (0.41s latency).
MAC Address: 18:34:51:F4:1D:6B (Apple)
Nmap scan report for 192.168.1.225
Host is up.
Nmap done: 256 IP addresses (9 hosts up) scanned in 2.63 seconds
```

And there we go, our Pi is located at `192.168.1.241`! Now we can ssh into it! The default username/password is `alarm/alarm` and root is `root/root`.

**NOTE!** The latest builds of Arch ARM come with OpenSSH 7.0+. This means that it will NOT allow you to connect as root over SSH. In order to change this, you should edit /etc/ssh/sshd_config with the following line:

```
PermitRootLogin yes
```

Here are the first things I did when I gained access to my Pi:

1. Upgraded the entire OS via pacman. Behold the power of Ach!
  ```
  [root@alarmpi ~]# pacman -Syu
  ```

2. Install `sudo`. I kept typing it without it being available :wink:
  ```
  [root@alarmpi ~]# pacman -S sudo
  ```

  This can take a little while, but note that it's a good idea to regularly do this.

3. Change root's password. It's not essential for hobbyist work at home, but it's good to get used to doing this on new Linux installs.
  ```
  [root@alarmpi ~]# passwd
  ```

4. The old vi that Arch ships with was acting really funky on me, so I installed vim.
  ```
  [alarm@alarmpi ~]# sudo pacman -S vim
  ```

5. Next up: let's change the timezone. First, have a look at the ones available:
  ```
  [alarm@alarmpi ~]# ls /usr/share/zoneinfo
  ```

  For example, my time zone is `/usr/share/zoneinfo/Europe/Brussels`. I need to remove the previous local timezone symlink and link it again:
  ```
  [alarm@alarmpi ~]# sudo rm /etc/localtime
  [alarm@alarmpi ~]# sudo ln -s /usr/share/zoneinfo/Europe/Brussels /etc/localtime
  ```

6. Now that the timezone has been taken care of, we should change the hostname of our little Pi buddy:
  First, edit the contents of `/etc/hostname` so it reflects your new hostname (I called my first Pi `docker-captain`, and the rest of them `docker-crewman`)
  Then, edit the `/etc/hosts` file so it looks like this:
  ```
  #
  # /etc/hosts: static lookup table for host names
  #

  #<ip-address>	<hostname.domain.org>	<hostname>
  127.0.0.1	localhost.localdomain	docker-captain
  ::1		localhost.localdomain	docker-captain
  ```

  **Now, rinse and repeat for every single Raspberry Pi you want to add to your mini-cluster!**
