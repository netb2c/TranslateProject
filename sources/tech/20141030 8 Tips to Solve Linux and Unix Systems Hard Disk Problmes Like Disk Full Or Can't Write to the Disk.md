Translating by ZTinoZ
8 Tips to Solve Linux & Unix Systems Hard Disk Problmes Like Disk Full Or Can’t Write to the Disk
================================================================================
Can't write to the hard disk on a Linux or Unix-like systems? Want to diagnose corrupt disk issues on a server? Want to find out why you are getting "disk full" messages on screen? Want to learn how to solve full/corrupt and failed disk issues. Try these eight tips to diagnose a Linux and Unix server hard disk drive problems.

![](http://s0.cyberciti.org/uploads/cms/2014/10/welcome-0-disk-problems.001.jpg)

### #1 - Error: No space left on device ###

When the Disk is full on Unix-like system you get an error message on screen. In this example, I'm running [fallocate command][1] and my system run out of disk space:

    $ fallocate -l 1G test4.img
    fallocate: test4.img: fallocate failed: No space left on device

The first step is to run the df command to find out information about total space and available space on a file system including partitions:

    $ df

OR try human readable output format:

    $ df -h

Sample outputs:

    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda6       117G   54G   57G  49% /
    udev            993M  4.0K  993M   1% /dev
    tmpfs           201M  264K  200M   1% /run
    none            5.0M     0  5.0M   0% /run/lock
    none           1002M     0 1002M   0% /run/shm
    /dev/sda1       1.8G  115M  1.6G   7% /boot
    /dev/sda7       4.7G  145M  4.4G   4% /tmp
    /dev/sda9       9.4G  628M  8.3G   7% /var
    /dev/sda8        94G  579M   89G   1% /ftpusers
    /dev/sda10      4.0G  4.0G     0 100% /ftpusers/tmp

From the df command output it is clear that /dev/sda10 has 4.0Gb of total space of which 4.0Gb is used.

#### Fixing problem when the disk is full ####

1.[Compress uncompressed log and other files][2] using gzip or bzip2 or tar command:

    gzip /ftpusers/tmp/*.log
    bzip2 /ftpusers/tmp/large.file.name

2.Delete [unwanted files using rm command][3] on a Unix-like system:

    m -rf /ftpusers/tmp/*.bmp

3.Move files to other [system or external hard disk using rsync command][4]:

    rsync --remove-source-files -azv /ftpusers/tmp/*.mov /mnt/usbdisk/
    rsync --remove-source-files -azv /ftpusers/tmp/*.mov server2:/path/to/dest/dir/

4.[Find out the largest directories or files eating disk space][5] on a Unix-like systesm:

    du -a /ftpusers/tmp | sort -n -r | head -n 10
    du -cks * | sort -rn | head

5.[Truncate a particular file][6]. This is useful for log file:

    truncate -s 0 /ftpusers/ftp.upload.log
    ### bash/sh etc ##
    >/ftpusers/ftp.upload.log
    ## perl ##
    perl -e'truncate "filename", LENGTH'

6.Find and remove large files that are open but have been deleted on Linux or Unix:

    ## Works on Linux/Unix/OSX/BSD etc ##
    lsof -nP | grep '(deleted)'
     
    ## Only works on Linux ##
    find /proc/*/fd -ls | grep  '(deleted)'

To truncate it:

     ## works on Linux/Unix/BSD/OSX etc all ##
    > "/path/to/the/deleted/file.name"
    ## works on Linux only ##
    > "/proc/PID-HERE/fd/FD-HERE"

### #2 - Is the file system is in read-only mode? ###

You may end up getting an error such as follows when you try to create a file or save a file:

    $ cat > file
    -bash: file: Read-only file system

Run mount command to find out if the file system is mounted in read-only mode:

    $ mount
    $ mount | grep '/ftpusers'

To fix this problem, simply remount the file system in read-write mode on a Linux based system:

    # mount -o remount,rw /ftpusers/tmp

Another example, from my [FreeBSD 9.x server to remount / in rw mode][7]:

    # mount -o rw /dev/ad0s1a /

### #3 - Am I running out of inodes? ###

Sometimes, df command reports that there is enough free space but system claims file-system is full. You need to check [for the inode][8] which identifies the file and its attributes on a file systems using the following command:

    $ df -i
    $ df -i /ftpusers/

Sample outputs:

    Filesystem      Inodes IUsed   IFree IUse% Mounted on
    /dev/sda8      6250496 11568 6238928    1% /ftpusers

So /ftpusers has 62,50,496 total inodes but only 11,568 are used. You are free to create another 62,38,928 files on /ftpusers partition. If 100% of your inodes are used, try the following options:

- Find unwanted files and delete or move to another server.
- Find unwanted large files and delete or move to another server.

### #4 - Is my hard drive is dying? ###

[I/O errors in log file (such as /var/log/messages) indicates][9] that something is wrong with the hard disk and it may be failing. You can check hard disk for errors using smartctl command, which is control and monitor utility for SMART disks under Linux and UNIX like operating systems. The syntax is:

    smartctl -a /dev/DEVICE
    # check for /dev/sda on a Linux server
    smartctl -a /dev/sda

You can also use "Disk Utility" to get the same information

[![](http://s0.cyberciti.org/uploads/l/tips/2007/07/500-GB-Hard-Disk-ATA-TOSHIBA-MK5061GSYF-dev-sda-%E2%80%94-Disk-Utility_014.png)][10]

Fig. 01: Gnome disk utility (Applications > System Tools > Disk Utility)

> **Note**: Don't expect too much from SMART tool. It may not work in some cases. Make backup on a regular basis.

### #5 - Is my hard drive and server is too hot? ###

High temperatures can cause server to function poorly. So you need to maintain the proper temperature of the server and disk. High temperatures can result into server shutdown or damage to file system and disk. [Use hddtemp or smartctl utility to find out the temperature of your hard on a Linux or Unix based system][11] by reading data from S.M.A.R.T. on drives that support this feature. Only modern hard drives have a temperature sensor. hddtemp supports reading S.M.A.R.T. information from SCSI drives too. hddtemp can work as simple command line tool or as a daemon to get information from all servers:

    hddtemp /dev/DISK
    hddtemp /dev/sg0

Sample outputs:

[![](http://s0.cyberciti.org/uploads/cms/2014/10/hddtemp-on-rhel-300x85.jpg)][12]

Fig.02: hddtemp in action

You can use the smartctl command as follows too:

    smartctl -d ata -A /dev/sda | grep -i temperature

#### How do I get the CPU temperature? ####

You can use Linux hardware monitoring tool such as [lm_sensor to get the cpu temperature on a Linux based][13] system:

    sensors

Sample outputs from Debian Linux server:

[![](http://s0.cyberciti.org/uploads/cms/2014/10/sensors-command-on-debian-server.jpg)][14]

Fig.03: sensors command providing cpu core temperature and other info on a Linux

### #6 - Dealing with corrupted file systems ###

File system on server may be get corrupted due to a hard reboot or some other error such as bad blocks. You can [repair corrupted file systems with the following fsck command][15]:

    umount /ftpusers
    fsck -y /dev/sda8

See [how to surviving a Linux filesystem failures][16] for more info.

### #7 - Dealing with software RAID on a Linux ###

To find the current status of a Linux software raid type the following command:

     ## get detail on /dev/md0 raid ##
    mdadm --detail /dev/md0
     
    ## Find status ##
    cat /proc/mdstat
    watch cat /proc/mdstat

Sample outputs:

[![](http://s0.cyberciti.org/uploads/cms/2014/10/linux-mdstat-output.jpg)][17]

Fig. 04: Find the status of a Linux software raid command

You need to replace a failed hard drive. You must u remove the correct failed drive. In this example, I'm going to replace /dev/sdb (2nd hard drive of RAID 6). It is not necessary to take the storage offline to repair the RAID on Linux. This only works if your server support hot-swappable hard disk:

    ## remove disk from an array md0 ##
    mdadm --manage /dev/md0 --fail /dev/sdb1
    mdadm --manage /dev/md0 --remove /dev/sdb1
     
    # Do the same steps again for rest of /dev/sdbX ##
    # Power down if not hot-swappable hard disk: ##
    shutdown -h now
     
    ## copy partition table from /dev/sda to newly replaced /dev/sdb ##
    sfdisk -d /dev/sda | sfdisk /dev/sdb
    fdisk -l
     
    ## Add it ##
    mdadm --manage /dev/md0 --add /dev/sdb1
    # do the same steps again for rest of /dev/sdbX ##
     
    # Now md0 will sync again. See it on screen ## 
    watch cat /proc/mdstat

See our [tips on increasing RAID sync speed on Linux][18] for more information.

### #8 - Dealing with hardware RAID ###

You can use the samrtctl command or vendor specific command to find out the status of RAID and disks in your controller:

    ## SCSI disk 
    smartctl -d scsi --all /dev/sgX
     
    ## Adaptec RAID array
    /usr/StorMan/arcconf getconfig 1
     
    ## 3ware RAID Array
    tw_cli /c0 show

See your vendor specific documentation to replace a failed disk.

### Monitoring disk health ###

See our previous tutorials:

1. [Monitoring hard disk health with smartd under Linux or UNIX operating systems][19]
1. [Shell script to watch the disk space][20]
1. [UNIX get an alert when disk is full][21]
1. [Monitor UNIX / Linux server disk space with a shell scrip][22]
1. [Perl script to monitor disk space and send an email][23]
1. [NAS backup server disk monitoring shell script][24]

### Conclusion ###

I hope these tips will help you troubleshoot system disk issue on a Linux/Unix based server. I also recommend implementing a good backup plan in order to have the ability to recover from disk failure, accidental file deletion, file corruption, or complete server destruction:

- [Debian / Ubuntu: Install Duplicity for encrypted backup in cloud][25]
- [HowTo: Backup MySQL databases, web server files to a FTP server automatically][26]
- [How To Set Red hat & CentOS Linux remote backup / snapshot server][27]
- [Debian / Ubuntu Linux install and configure remote filesystem snapshot with rsnapshot incremental backup utility][28]
- [Linux Tape backup with mt And tar command tutorial][29]

--------------------------------------------------------------------------------

via: http://www.cyberciti.biz/datacenter/linux-unix-bsd-osx-cannot-write-to-hard-disk/

作者：[nixCraft][a]
译者：[译者ID](https://github.com/译者ID)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创翻译，[Linux中国](http://linux.cn/) 荣誉推出

[a]:http://www.cyberciti.biz/tips/about-us
[1]:http://www.cyberciti.biz/faq/howto-create-lage-files-with-dd-command/
[2]:http://www.cyberciti.biz/howto/question/general/compress-file-unix-linux-cheat-sheet.php
[3]:http://www.cyberciti.biz/faq/howto-linux-unix-delete-remove-file/
[4]:http://www.cyberciti.biz/faq/linux-unix-bsd-appleosx-rsync-delete-file-after-transfer/
[5]:http://www.cyberciti.biz/faq/how-do-i-find-the-largest-filesdirectories-on-a-linuxunixbsd-filesystem/
[6]:http://www.cyberciti.biz/faq/truncate-large-text-file-in-unix-linux/
[7]:http://www.cyberciti.biz/faq/howto-freebsd-remount-partition/
[8]:http://www.cyberciti.biz/tips/understanding-unixlinux-filesystem-inodes.html
[9]:http://www.cyberciti.biz/tips/linux-find-out-if-harddisk-failing.html
[10]:http://www.cyberciti.biz/tips/linux-find-out-if-harddisk-failing.html
[11]:http://www.cyberciti.biz/tips/howto-monitor-hard-drive-temperature.html
[12]:http://www.cyberciti.biz/datacenter/linux-unix-bsd-osx-cannot-write-to-hard-disk/attachment/hddtemp-on-rhel/
[13]:http://www.cyberciti.biz/faq/howto-linux-get-sensors-information/
[14]:http://www.cyberciti.biz/datacenter/linux-unix-bsd-osx-cannot-write-to-hard-disk/attachment/sensors-command-on-debian-server/
[15]:http://www.cyberciti.biz/tips/repairing-linux-ext2-or-ext3-file-system.html
[16]:http://www.cyberciti.biz/tips/surviving-a-linux-filesystem-failures.html
[17]:http://www.cyberciti.biz/datacenter/linux-unix-bsd-osx-cannot-write-to-hard-disk/attachment/linux-mdstat-output/
[18]:http://www.cyberciti.biz/tips/linux-raid-increase-resync-rebuild-speed.html
[19]:http://www.cyberciti.biz/tips/monitoring-hard-disk-health-with-smartd-under-linux-or-unix-operating-systems.html
[20]:http://www.cyberciti.biz/tips/shell-script-to-watch-the-disk-space.html
[21]:http://www.cyberciti.biz/faq/mac-osx-unix-get-an-alert-when-my-disk-is-full/
[22]:http://bash.cyberciti.biz/monitoring/shell-script-monitor-unix-linux-diskspace/
[23]:http://www.cyberciti.biz/tips/howto-write-perl-script-to-monitor-disk-space.html
[24]:http://bash.cyberciti.biz/backup/monitor-nas-server-unix-linux-shell-script/
[25]:http://www.cyberciti.biz/faq/duplicity-installation-configuration-on-debian-ubuntu-linux/
[26]:http://www.cyberciti.biz/tips/how-to-backup-mysql-databases-web-server-files-to-a-ftp-server-automatically.html
[27]:http://www.cyberciti.biz/faq/redhat-cetos-linux-remote-backup-snapshot-server/
[28]:http://www.cyberciti.biz/faq/linux-rsnapshot-backup-howto/
[29]:http://www.cyberciti.biz/faq/linux-tape-backup-with-mt-and-tar-command-howto/
