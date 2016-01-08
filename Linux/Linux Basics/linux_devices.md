udev system
  Enbables user-space programs the ability to automatically configure and use new devices.

device files (aka device nodes)
  Are the files that represent devices

  Important note: Not all devices have device files because the block and character device I/O interfaces are not for all use cases.  
    Ex: network interfaces don't have device files.
      While it would be theoretically possible to interact with a network interface using a single character device; it would be very difficult.  As such, the kernel uses other I/O interfaces.

  In general. it's easy to manipulate most devices on a Unix system because the kernel presents many of the device I/O interfaces to processes as files (aka device nodes)
    Although, these interfaces can be somewhat limited (relative to what the device can truly do.)

/dev
  /dev/null
    The kernel controls this device, and will ignore any input written to it.

  Important: The kernel assigns devices in the this directory in the order in which they are found, thus, a device may have a different name between reboots.
    For an alternative solution, see sysfs (below) Also see sysfs and the differences between /dev (below)

  ls -la and devices
    For a device, the first character of a files mode will be either b,c,p,s
      b-block device
        Programs access data from a block device in chunks.  A disk device is a type of block device.
          Disk devices
            Disks can be easily split up into blocks of data.  Because their size is fixed (and thus easily indexible), processes have random access to any block in the device with the help of the kernel.
      c-character device 
        Ex: /dev/null, printers directly attached to your computer
        Character devices work with data streams.  You can only read characters from (or write characters to) character devices.  These devices don't have a size and it's the kernel who usually performs the read/write operation on the device.

        When a character device is being interacted with, the kernel can't back up and reexamine the data stream after it has passed data to a device or process.
      p-pipe device
        A 'named pipe' or 'pipe device' is like a character device.
        However, instead of a kernel driver on the other end of the I/O stream, it's another process.
      s-socket device
        Sockets are heavily used in interprocess communication and are frequently found outside /dev

        Socket files represent Unix domain sockets.

    Major/minor device numbers
      When you ls -l a device you'll see 2 comma delimited numbers before the time.
        This number helps the kernel identify a device.

        Similar devices usually have the same number
          Ex: sda3, sda1 have the same number (as they're both hard disk partitions)
sysfs
  Found is /sys

  Provides a uniform view of attached devices based on their hardware attributes through a system of files and directories.  

  /sys/block
    Sym links to all of the block devices

  sysfs is much more granular relative to /dev
    A /dev file is used so a user process can use the device, whereas the /sys/devices path is used to view information and manage the device.

    Ex: A SATA hard disk at /dev/sda might have the path in sysfs: /sys/devices/pci0000:00/0000:00:1f.2/host0/target0:0:0/0:0:0:0/block/sda
      While you can't rely on /dev/sda to consistently map to the write device, you can with sysfs

      By going into a directory (like that of above) you can see pretty granular file names that correlate to different meta-data (or other parts) of a device.

  udevadm
  To find the sysfs location of a device in /dev
    Ex: udevadm info --query=all --name=/dev/sda

dd
  A program that is very useful when working with block and character devices.
    Warning: It is very powerful.  It often helps to write the output to a new file if you're not sure what it is going to do.  Otherwise, you might easily corrupt files and data on a device.

  Sole function is to read from an input file or stream and write to an output file or stream.
    It also can do some encoding in this process.

  dd copies data in blocks of a fixes suze.

  Ex: use dd with a character device and some common options
    dd if=/dev/zero of=new_file bs=1024 count=1

      This copies a single 1024-byte block from /dev/zero to new_file.  As an aside (/dev/zero) is a continuous stream of zero bytes.

      Options
        if=file
          This is the input file.  The default is STDIN
        of=file
          The output file.  The default is STDOUT
        bs=size
          The block size.  dd reads and writes this many bytes of data at a time.  To abbreviate large chunks of data:
            b=512
            k=1024
              Ex: bs=1k

        ibs=size, obs=size
          The input and output block sizes (respectively)

          If you can use the same block size for both input and output; us the bs option.

        skip=num
          Skip past the first num blocks in the input file/stream and do not copy them to the output

        count=num
          The total number of blocks to copy.
            You can rate limit long streams and huge files this way.

            You can utilize count and skip together to selectively copy certain parts of a file.

            You can utilize count 







