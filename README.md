# FlashFloppy images for Atari ST

This repository contains floppy disk images for use on a Gotek with FlashFloppy on an Atari ST(E).

Using the images and config provided, you will be able to access High Density 1440 kb (HD), Extra-high density 2880 kb (ED) and custom disks with sizes up to 14 mb on a stock Atari ST (without high density support). The images are formatted using PC compatible FAT12 formatting.

Tested on TOS 1.04 and 2.06 using FlashFloppy 3.38 in native mode using a Gotek with an AT32F435 controller.

## How to use these images?

The images are preformatted, all you need to do is make a copy of the images to the USB stick of your Gotek with FlashFloppy.

A config file is needed to make FlashFloppy understand how to deal with these images. Copy IMG.CFG to the USB stick of your Gotek. The exact location of the files on your Gotek probably depends on the FlashFloppy mode used. For more information, see: https://github.com/keirf/flashfloppy/wiki/Initial-Setup

The HD image has about half the random access speed of a DD floppy. The ED image has about quarter the random access speed of a DD floppy. The technical reason for this is explained in the text below.

The custom XL images start with approximately DD random access speed for the smallest size and get increasingly slower as the size increases. The largest image sizes are probably only useful for backups and transferring large amounts of data between computers.

Use the HD and ED images only if you need the image to be compatible with standard HD and ED format. If all you need is a larger floppy image, then use the custom XL size images instead as they are generally faster. Don't use the 14 mb custom XL image if you require compatibility with the Hatari emulator.

Please note, the larger image sizes sometimes cause read errors, especially after swapping disks. Retry will normally solve the problem.

## How does it work?

A floppy disk is divided into cylinders or tracks (circles on the disk), sectors (part of those circles containing 512 bytes each) and sides (2 for double sided disks). DD, HD and ED disks all use the same amount of cylinders (usually 80).

The amount of sectors that fit into one cylinder is determined by the data rate, the amount of bits written to the disk per second.

A 3.5" floppy drive rotates the disk at 300 rpm. After reading all the sectors in one cylinder, it moves the head to the next cylinder using a step.

The specifications of the different types of disks are as follows:
- DD: 720 kb, 80 cylinders, 9 sectors and 2 sides using a data rate of 250 kbit/s 
- HD: 1440 kb, 80 cylinders, 18 sectors and 2 sides using a data rate of 500 kbit/s
- ED: 2880 kb, 80 cylinders, 36 sectors and 2 sides using a data rate of 1000 kbit/s

This means that normal HD and ED floppy drives have 2 and 4 times the speed of a DD drive.

A stock Atari ST can only access floppy disks using a 250 kbit/s data rate. To access regular HD floppies, you need a floppy controller that supports 500 kbit/s, which is only available on the Mega STE / TT / Falcon. It is possible to modify other STs to add HD support, but that involves soldering and possibly even swapping out the floppy disk controller.

When using a Gotek, the hardware of a floppy disk and drive are emulated. By default it will adjust the data rate according to the disk size. But because it's emulated, it allows doing some tricks with which you can read HD and ED floppies without modifying your Atari ST.

One trick is to reduce the (emulated) rotation speed to allow more sectors to fit in one cylinder. For HD this means reducing to 50% (150 rpm) and for ED 25% (75 rpm). This comes at a price however.

There are 2 aspects to the speed of a floppy drive. One is the data rate, which determines the speed at which the bits are read. The other is the seek time. If the computer wants to access a certain part of floppy, it first needs to step the head to the correct cylinder, each step takes a small amount of time. When it arrives at the correct cylinder, the computer needs to wait until the rotation reaches the sector it is looking for.

Since the Atari ST has a fixed data rate, the data transfer speed will be the same for all types of floppies read via the Gotek, regardless of the rotation speed. Reducing the rotation speed will slow down the seek times however. For example when reducing the rotation speed to 25% to be able to read an ED disk will result in 4 times slower seek times.

There's a second way to get more data with the same data rate however: increasing the number of cylinders. Normal floppies have hardware limitations restricting the floppy drive to just over 80 cylinders. FlashFloppy supports up to 255 cylinders. So instead of reducing the rotation speed, we can simply increase the amount of cylinders to be able to store more data without a big performance penalty.

By combining these two tricks we can create even larger floppies. As explained, the more the rotation speed is decreased, the slower the disk access. Reducing it too much eventually results in the computer giving up the seek and result in a disk error.

## Accessing image data on your PC

On Ubuntu Linux, you can access the data in a floppy image read-only by double clicking on the image. To access the data read/write, you will have to mount it manually with a command such as:
```bash
mkdir floppy
sudo mount -o uid=1000,gid=1000 floppy.img floppy
```
Replace 1000 with the uid and gid of your linux user (the initial user after installation has uid/gid 1000).

You can also access the floppy images by mounting them in an emulator such as Hatari. Before mounting them in Hatari, you need to rename the file extension to ".st". Hatari does not seem to support the 13770 kb floppy size however. Don't forget to rename the file extension back to ".img" before using the image on the Gotek, or you will get an error on the Gotek.

## Generating new images

A generate Bash script (for use on Linux or Windows WSL) is added for generating the disk images. The math works as follows:

Each sector is 512 bytes, but there are 2 sides, so simply count each sector as 1 kbyte.

A DD 300 rpm cylinder fits 9 sectors with standard formatting. If you want to fit more sectors, calculate the RPM using: 300 * 9 / desired sectors. Don't go below 50 rpm, you will most likely get read errors and disk access will be unbearably slow.

If increasing the disk size, first try to increase the amount of cylinders as that has a smaller performance impact.

The final disk image size in kbyte is cylinders * sectors.

For more information about IMG.CFG, see: https://github.com/keirf/flashfloppy/wiki/IMG.CFG-Configuration-File

## License

This work is published under the CC0 license: https://creativecommons.org/share-your-work/public-domain/cc0/

Attribution by linking to this repository is of course appreciated.
