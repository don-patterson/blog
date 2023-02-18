# Idea
I think I want to make a blog. I'm not sure if anyone will read it, or whether
I want them to, but I want to start writing down things I've done so I can
refer back to it, or share it, etc.

Also, I'm looking for an exuse to learn about revealjs. It's probably not the
best platform for blog posts, but at the very least it might force this to be
a bit instructional.
***
Today I want to write about how I went down a rabbit hole to install Windows on
my ancient desktop. It has an Nvidia GTX-555 graphics card, which has one of
those mini-HDMI ports. Ubuntu supports it sometimes, but every once in a while
an update happens and either audio goes away, video goes away, or both. I've
been able to trudge through forums, stack overflows, and github issues to
eventually get to the bottom of it each time, except for the latest one. It
cost me too many evenings (I only get like an hour and a half of free time to
watch a show or play a game, so scouring ubuntu forums for 45 minutes and
trying a bunch of random apt-get installs isn't that appealing!).
***
Anyway, a random Youtube video saved the day: https://www.youtube.com/watch?v=Y388W8MaPME

Basically you have to take the windows iso image, and reformat it yourself so
that your computer can recognize it as bootable. He did it with `gparted`, but
I was doing it on a headless raspberry pi, so here's how you can do it with
regular `parted`. 

Rough notes below. I'll clean this up so it is separated into logical chunks.
***
## Let's see how giant code blocks look:
```
don@pi3:~ $ sudo parted /dev/sda
GNU Parted 3.4
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
Model: Lexar USB Flash Drive (scsi)
Disk /dev/sda: 32.0GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      17.4kB  1024MB  1024MB  fat32        BOOT     msftdata
 2      1024MB  32.0GB  31.0GB  ntfs         INSTALL  msftdata

mklabel (gpt)
mkpart (fat32, BOOT)
mkpart (ntfs, INSTALL)
quit


sudo mkfs.vfat /dev/sda1 # immediate
sudo mkfs.ntfs /dev/sda2 # took like an hour!

copy everything but sources into boot, then copy sources/boot.wim
copy everything into install





## tried it on another usb
don@pi3:~ $ sudo parted /dev/sda
GNU Parted 3.4
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print                                                            
Model: SanDisk Cruzer (scsi)
Disk /dev/sda: 8029MB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags: 

Number  Start  End     Size    File system  Flags
 1      0.00B  8029MB  8029MB  fat32

(parted) rm 1                                                             
(parted) mklabel
New disk label type? gpt                                                  
Warning: The existing disk label on /dev/sda will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? Yes                                                               
(parted) print                                                            
Model: SanDisk Cruzer (scsi)
Disk /dev/sda: 8029MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start  End  Size  File system  Name  Flags

(parted) mkpart                                                           
Partition name?  []? BOOT                                                 
File system type?  [ext2]? fat32                                          
Start? 0%                                                                 
End? 1024MB                                                               
(parted) mkpart                                                           
Partition name?  []? INSTALL                                              
File system type?  [ext2]? ntfs
Start? 1024MB                                                             
End? 100%                                                                 
(parted) print
Model: SanDisk Cruzer (scsi)
Disk /dev/sda: 8029MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  1024MB  1023MB  fat32        BOOT
 2      1024MB  8029MB  7004MB  ntfs         INSTALL

(parted) quit                                                             
Information: You may need to update /etc/fstab.



don@pi3:~ $ ls /dev/sda*
/dev/sda  /dev/sda1  /dev/sda2
don@pi3:~ $ sudo mkfs.vfat /dev/sda1
mkfs.fat 4.2 (2021-01-31)
don@pi3:~ $ sudo mkfs.ntfs /dev/sda2
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes:  27%
... a year later (15 minutes for 8G)
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.



# then I had to mount the iso and usb
don@pi3:~ $ sudo mkdir boot install win
don@pi3:~ $ sudo mount /dev/sda1 boot
don@pi3:~ $ sudo mount /dev/sda2 install
don@pi3:~ $ sudo mount -o loop Win10_22H2_English_x64.iso win
# failed, googled some stuff
apt search isofs
sudo apt install libisofs6
don@pi3:~ $ sudo mount -o loop Win10_22H2_English_x64.iso win
mount: /home/don/win: WARNING: source write-protected, mounted read-only.

TIL about ls --hide=thing

try this and see if it looks right:
echo cp -r $(ls --hide=sources) ../boot
and if so, 
cp -r $(ls --hide=sources) ../boot

then
mkdir ../boot/sources
cp sources/boot.wim ../boot/sources/


finally:
cp -r * ../install


done!

```
<!-- .element: class="r-stretch" -->

ooh, not bad