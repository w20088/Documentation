# Recovery Android to the Poplar Board

## Create a USB drive for flashing

To allow recovery of a Poplar board in a "bricked" state, or to flash the Poplar board with an Android image,  prepare a USB flash drive.

### Step 1: Identify your USB flash drive device

  Insert the USB flash drive into your host system, and identify
  your USB device:

```shell
    grep . /sys/class/block/sd?/device/model
```
  If you recognize the model name as your USB flash device, then
  you know which "sd" device to use.  Here's an example:

```shell
    /sys/class/block/sdh/device/model:Patriot Memory
                     ^^^
```
  I had a Patriot Memory USB flash drive, and the device name
  I'll want is "/dev/sdh" (based on "sdh" above).  Record this name:

```shell
    USBDISK=/dev/sdh    # Make sure this is *your* device
```

  The instructions that follow assume your USB flash drive needs to be
  formatted "from scratch."  Once formatted, all that's required is to
  copy "fastboot.bin" to the first partition on the drive, and then
  properly eject the medium before removing the USB drive.

### Step 2: Format the flash drive using MBR partitioning.

  THIS IS VERY IMPORTANT.  The following commands will COMPLETELY
  ERASE the contents of whatever device you specify here.  So be
  sure USBDISK defines the flash device you intend to erase.

  You will need superuser access.  First, unmount anything mounted
  on that device:

```shell
    sudo umount ${USBDISK}?
```

  Next, clobber any existing partitioning information that might be
  found at the beginning of the device:

```
    sudo dd if=/dev/zero of=${USBDISK} bs=2M count=1 status=none
```

  Create a DOS MBR partition table on the USB flash drive with a
  single partition, and format that partition using FAT32.

```shell
    {   echo label:dos
    echo 1: start=8 size=62496KiB type=0x0c
    echo write
    } | sudo sfdisk --label dos ${USBDISK}
    sudo mkfs.fat -F 32 ${USBDISK}1
```

You now have a properly formated USB drive that is ready to be used for flashing the recovery files and Android Images to a Poplar board.

## Installing partition table and bootloader

### Step 1: Download partition table recovery files and copy them to the USB disk created previously (see [Create a USB drive for flashing](#create-a-usb-drive-for-flashing))

```
git clone https://github.com/96boards-poplar/l-loader.git
cp -r l-loader/installer/*  ${your_usb_mount_point}
sync
```

* An USB automount point on Ubuntu: /media/username/631B-5041

### Step 2: Install partition table

Following [instruction here](#put-board-in-recovery-or-flashing-state) to put board in recovery or flashing state:

```
usb reset
fatload usb 0:1 ${scriptaddr} recovery_files/install.scr
source ${scriptaddr}
```

### Step 3: Install bootloader

Right after partition table installation, you can choose to install bootloader built from source by yourself, or prebuilt ones in l-loader/prebuilts/, using fastboot.

Device side, U-Boot command:

```
poplar# fastboot 0
```

Host side, fastboot command:

```
$ fastboot flash mmcsda1 loader.bin
```

In addition to bootloader installation, you also have to setup the correct u-boot bootcmd, so it can auto boot the android once the images are flashed.

```
env set bootcmd run bootai
env save
```

## Put board in recovery or flashing state

  If a Poplar board is in a "bricked" state, it can be booted using
  the USB flash drive prepared above.

### Step 1: Prepare the Poplar board for power-on

- The Poplar board should be powered off.  You should have a cable
  from the Poplar's micro USB based serial port to your host
  system so you can connect and observe activity on the serial port.
  For me, the board console shows up as /dev/ttyUSB0 when the USB
  cable is connected.  The serial port runs at 115200 baud.  I use
  this command to see what's on the console:

```shell
      screen /dev/ttyUSB0 115200
```

### Step 2: Insert the USB flash drive on the Poplar board

- There are a total of 4 USB connectors on the Poplar board.  Two
  are USB 2.0 ports, they're stacked on top of each other.  Insert
  the USB memory stick into one of these two.

- There is a "USB_BOOT" button on the board.  It is one of two
  buttons on same side of the boards as the stacked USB 2.0 ports.
  To boot from the memory stick, this button needs to be depressed
  at power-on.  You only need to hold it for about a second;
  keeping it down a bit longer does no harm.

- Next you will be powering on the board, but you need to interrupt
  the automated boot process.  To do this, be prepared to press a
  key, perhaps repeatedly, in the serial console window until you
  find the boot process has stopped.

### Step 3: Boot the Poplar board from the USB flash drive

- Power on the Poplar board (while pressing the USB_BOOT button),
  and interrupt its automated boot with a key press.  This should
  lead to a "poplar# " prompt.
