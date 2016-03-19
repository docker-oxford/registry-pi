# pi-registry

Docker Registry on a Raspberry Pi.

## DNS

Follow the setup in [local-registry](https://github.com/docker-oxford/local-registry) for keys and a trusted DNS.

If you don't set up DNS and real certificates, you can still use your registry, but you'll have to tell everyone using it to add `--insecure-registry ip.or.hostname:port` to  `/etc/defaults/docker` and restart the Docker Engine. That will completely disable security for any interaction with your registry, and may (will) cause (various) problems with authentication later on.

## OS

The Hypriot OS lets you run Docker on the pi.

## Registry

We will build the registry for arm with `hypriot/rpi-golang` as the base image.

## Web server

We'll use the web server setup from the [official documentation](https://docs.docker.com/registry/apache/), but alter it slightly to work on pi.

# Setting it up

## OS installation

Tested on Mac. The steps will be similar on Linux (PRs welcome for Linux steps).

    ## NOTE: You are responsible for your own data. This tutorial assumes that you have made recent backups of your laptop and other devices, that you are using a blank SD card or are happy to erase all data on it. The author does not take responsibility for any data loss incurred by following any advice given in this document.

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
    # WARNING: If you misuse the dd command you can end up destroying your system and lose data
    # Double-check this command twice and make sure that of=/dev/rdisk5 corresponds to your SD card.
    # dd will not warn you before overwriting your boot partition. It will do exactly what you tell it to. Silently.
    #
    # if = input file (where to read from)
    # of = output file (where to write to)
    # bs = block size in bytes
    #
    # Here we read from the image file and write directly to the (unbuffered) block device /dev/rdisk5
    sudo dd if=hypriot-rpi-20160306-192317.img of=/dev/rdisk5 bs=524288

    # Insert the SD card into your Raspberry Pi and plug the Pi with an Ethernet cable to your router. It needs a DHCP server to get an IP address.
    # When the Pi has booted, find its IP from the router's list of connected hosts.

The default SSH username is `root` with password `hypriot`.

## Pi configuration (optional)

You need [Ansible](http://www.ansible.com) installed for this. You can skip this step, but doing this will make it easier and more secure to interact with the pi. If you skip this step, at least change the default root password and disable root SSH access.

Instructions in the [hypriot-pi-setup readme](hypriot-pi-setup/readme.md).

## Build Dockerimages

Registry

    # SCP over the modified Dockerfile
    scp rpi-registry-Dockerfile 192.168.0.123:

    # Clone the distribution repo
    git clone https://github.com/docker/distribution.git

    # Update the Dockerfile
    mv rpi-registry-Dockerfile distribution/Dockerfile

    # Build the registry Dockerimage
    cd distribution
    docker build --tag rpi-registry .

    # Check your images
    docker images
    REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
    rpi-registry         latest              5546c77ac10a        3 hours ago         777.6 MB
    hypriot/rpi-swarm    latest              c298de062190        13 days ago         13.27 MB
    hypriot/rpi-golang   tar-1.5.2           a1254db987ac        12 weeks ago        408.6 MB

Webserver

    # Copy over the files from rpi-httpd
    rsync -rv rpi-httpd 192.168.0.123:

    # SSH in
    ssh 192.168.0.123

    # Become root and build Dockerimage
    sudo su -
    cd /home/yourusername/rpi-httpd #replace yourusername
    docker build --tag rpi-httpd:2.4 .
