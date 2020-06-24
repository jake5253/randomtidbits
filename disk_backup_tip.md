**Washing the partition(s)**
If the disk is in good working condition, you will get better compression if you wash the empty space on the disk with zeros. If the disk is failing, skip this step.

If you're imaging an entire disk then you will want to wash each of the partitions on the disk.

**CAUTION**: Be careful, you want to set the _of=_ to a file in the mounted partition, **NOT THE PARTITION ITSELF**!

```mkdir image_source
sudo mount /dev/sda1 image_source
dd if=/dev/zero of=image_source/wash.tmp bs=4M
rm image_source/wash.tmp
sudo umount image_source
```

**Making a Partition Image**
```mkdir image
sudo dd if=/dev/sda1 of=image/sda1_backup.img bs=4M
```
Where _sda_ is the name of the device, and _1_ is the partition number. Adjust accordingly for your system if you want to image a different device or partition.

**Making a Whole Disk Image**

```mkdir image
sudo dd if=/dev/sda of=image/sda_backup.img bs=4M
```

Where _sda_ is the name of the device. Adjust accordingly for your system if you want to image a different device.

**Compression**
Make a "_squashfs_" image that contains the full uncompressed image.

```sudo apt-get install squashfs-tools
mksquashfs image squash.img
```

**Streaming Compression**
To avoid making a separate temporary file the full size of the disk, you can stream into a squashfs image.

```mkdir empty-dir
mksquashfs empty-dir squash.img -p 'sda_backup.img f 444 root root dd if=/dev/sda bs=4M'
```

* If you already have backup.img.gz, you can avoid exracting a huge temporary image before moving it into a squash by extracting it directly into the squash image 
`sudo mksquashfs image-dir /path/of/new/compressed/squash.img -p 'sda_image_inside_squash.img f 444 root root gzip -dc /path/to/existing/backup.img.gz'
`


**Mounting a compressed partition image**
First mount the squashfs image, then mount the partition image stored in the mounted squashfs image.

```mkdir squash_mount
sudo mount squash.img squash_mount
```

Now you have the compressed image mounted, mount the image itself (that is inside the squashfs image)

```mkdir compressed_image
sudo mount squash_mount/sda1_backup.img compressed_image
```

Now your image is mounted under compressed_image.


If you wanted to simply restore the disk image onto a partition at this point (instead of mounting it to browse/read the contents), dd the image at squash_mount/sda1_backup.img onto the destination instead of mounting.

`sudo dd if=squash_mount/sda1_backup.img of=/dev/sda1`


**Mounting a compressed full disk image**
This requires you to use a package called kpartx. kpartx allows you to mount individual partitions in a full disk image.

`sudo apt-get install kpartx`


First, mount your squashed partition that contains the full disk image

```mkdir compressed_image
sudo mount squash.img compressed_image
```


Now you need to create devices for each of the partitions in the full disk image:

`sudo kpartx -a compressed_image/sda_backup.img`


This will create devices for the partitions in the full disk image at _/dev/mapper/loopNpP_ where _N_ is the number assigned for the loopback device, and _P_ is the partition number. For example: /dev/mapper/loop0p1.


Now you have a way to mount the individual partitions in the full disk image:

```mkdir fulldisk_part1
sudo mount /dev/mapper/loop0p1 fulldisk_part1
```
