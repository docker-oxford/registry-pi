# pi-registry

Docker Registry on Raspberry Pi.

## DNS

Follow the setup in [local-registry](https://github.com/docker-oxford/local-registry) for keys and a trusted DNS.

If you don't set it up, you can still use your registry, but you'll have to tell everyone using it to add `--insecure-registry ip.or.hostname:port` to  `/etc/defaults/docker` and restart the Docker Engine.

## OS

The Hypriot OS lets you run Docker on the pi.

## Registry

We will build the registry for arm with `hypriot/rpi-golang` as the base image.

## Web server

We'll use the web server setup from the [official documentation](https://docs.docker.com/registry/apache/), but alter it slightly to work on pi.

# Setting it up

## OS installation

Tested on Mac. The steps will be similar on Linux (PRs welcome for Linux steps).

    ## NOTE: You are responsible for your own data. This tutorial assumes that you have made recent backups of your laptop and other devices, that you are using a blank SD card or are happy to erase the data on it. I will in no way be held responsible for any data loss incurred by following these steps.

    # Download Hypriot OS
    wget https://downloads.hypriot.com/hypriot-rpi-20160306-192317.img.zip

    # Unzip image file
    unzip hypriot-rpi-20160306-192317.img.zip

    # Insert an SD card
    # List all disks
    diskutil list
    /dev/disk0 (internal, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:      GUID_partition_scheme                         251.0 GB   disk0
       1:                        EFI EFI                     209.7 MB   disk0s1
       2:          Apple_CoreStorage Macintosh HD            250.1 GB   disk0s2
       3:                 Apple_Boot Recovery HD             650.1 MB   disk0s3
    /dev/disk1 (internal, virtual):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:                  Apple_HFS Macintosh HD           +249.8 GB   disk1
                                     Logical Volume on disk0s2
                                     12345678-ABCD-1234-EFAB-1234567890AB
                                     Unlocked Encrypted
    /dev/disk5 (internal, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:     FDisk_partition_scheme                         31.1 GB    disk5
       1:             Windows_FAT_16 RECOVERY                1.2 GB     disk5s1
       2:                      Linux                         33.6 MB    disk5s5
       3:             Windows_FAT_32 boot                    66.1 MB    disk5s6
       4:                      Linux                         29.8 GB    disk5s7

    # Erase the SD card
    # NOTE: All data on the SD card will be lost.
    diskutil eraseDisk fat32 %noformat% disk5

    # Unmount the SD card
    diskutil unmountDisk /dev/disk5

    # Write image to SD card
    # WARNING: If you misuse dd you can end up destroying your system and lose data
    # Double-check this command twice and make sure that of=/dev/rdisk5 corresponds to your SD card.
    # dd won't ask you if you really want to write to your boot partition, it will do exactly what you tell it to. Silently.
    #
    # if = input file (where to read from)
    # of = output file (where to write to)
    # bs = block size in bytes
    #
    # Here we read from the image file and write directly to the (unbuffered) block device /dev/rdisk5
    sudo dd if=hypriot-rpi-20160306-192317.img of=/dev/rdisk5 bs=524288

    # Insert the SD card into your Raspberry Pi and plug the Pi with a cable to your router. It needs a DHCP server to get an IP address.
    # When the Pi has booted, find its IP from the router's list of connected hosts.

## Pi configuration (OPTIONAL)

You need [Ansible](http://www.ansible.com) installed for this. You can skip this step, but it will make it easier and more secure to interact with the pi. If you skip this, at least change the default root password and disable root SSH access.

    Follow the steps in the [readme](setup/readme.md)
