---
layout: post
title: "Transporter internals"
date: 2014-05-11 20:38:39 -0400
comments: true
categories: 
---
I recently got a [Transporter](http://www.filetransporter.com/) Sync after having heard about it on [Accidental Tech Podcast](http://atp.fm/).  The [Sync](http://www.filetransporter.com/products/transporter-sync/) comes without an internal hard drive, and you plug one in via USB.  I'm using a Seagate external that was cheap at Costco.  Unfortunately, the Transporter and the Seagate use barrel plugs for their power that are nearly the same size.  I learned this after having to move the pair and accidentaly pluggin the Seagate's 12 volt power supply into the Transporter, which only wants 5 volts.  Thankfully, Transporter is a *super* standup company and, despite this being unarguably my fault, sent me a new Sync free-of-charge when I told them of my error.  That's a big check in the customer support column.

While I had the drive that was connected to the Transporter sitting idle, I figured I'd peruse its contents.  When you plug a drive into a Transporter Sync it is wiped, and the Transporter puts…well, it uses the drive however it sees fit.  Transporter makes this very clear, with a sticker covering the USB port making it clear that whatever drive you plug in will be completely erased.

The disk has four partitions: the first two are 1 GB each, the third is 2 GB, and the last is the remainder fo your disk.  Partition 1 is a pretty standard Linux affair:

    .rnd    etc/           linuxrc       opt/          sys/         var/
    bin/    home/          lost+found/   proc/         tftpboot/
    boot/   installation/  mnt/          replicator/   tmp/
    dev/    lib/           old-root/     sbin/         usr/

file(1) says that the second is "data", nothing more (even with the [--special-files](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/file.1.html) option); given its size, I'm guessing swap, despite not being metioned in the first partition's fstab.  The third partition is mounted as `/opt` and contains just a few directories:

    core/   log/   lost+found/   ntp/   tmp/   upgrade-manager/

The `log` dir has logs for network interfaces (ifplugd, eth0-dhcp, wifi) and Samba.  The `ntp` dir just has a drift file, and the `upgrade-manager` has a logs, presumably about upgrades.

Finally, the big partition is mounted as `/replicator` and contains the data you store, among other things:

    configuration/ diskInfo.log  logs/          samba/         staging/       sys/
    directories/   history/      lost+found/    shadowPools/   storagePools/  uuids/

The `history` and `logs` directories are sizeable, tens of megabytes, with the former containing some metadata about syncing and the latter containg logs about syncing.  Both `shadowPools` and `storagePools` contain a directory that is a UUID where your data is stored.  `df` on my Mac doesn't tell me anything about the partition's capacity or usage (because it's mounted using [ext4fuse](https://github.com/gerard/ext4fuse) via [fuse4x](http://fuse4x.github.io/)), so I can't say for sure, but I figure some sort of hard link shenanigans is happening between those two directories.

Descending into either of those UUID-named directories you'll find the files that you have in the `~/Transporter` directory on your computer.  There is also a `.RootOfNetworkAttached` directory, that is the contents of your Transporter Library.  The first thing is actually what I was trying to ensure—that the files synced by Transporter à la Dropbox also exist on the Transporter.
