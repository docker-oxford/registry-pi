# registry-pi

Docker Registry on a Raspberry Pi.

Tested on Raspberry Pi 3.

## Background

If you have a group of 40 or more people in one room with limited bandwidth, everyone trying to pull Dockerimages from Docker Hub, you are going to have a bad time.

You can set up a registry proxy or a private registry locally to alleviate the network. This can be done relatively easily on a laptop, so why bother with a Raspberry Pi?

* Portable (you can keep it in your jacket pocket)
* Energy efficient (runs on 5 V so you can use a regular phone charger)
* Low cost (you don't have to buy a $300 laptop)
* It's fun :smile:

## Overview

### DNS

Follow the setup in [local-registry](https://github.com/docker-oxford/local-registry) for keys and a trusted DNS.

If you don't set up DNS and real certificates, you can still use your registry, but you'll have to tell everyone using it to add `--insecure-registry ip.or.hostname:port` to  `/etc/defaults/docker` and restart the Docker Engine. That will completely disable security for any interaction with your registry, and may (will) cause (various) problems with authentication later on. This guide assumes that you have a real DNS name and a signed certificate to prove it.

### OS

[Hypriot OS](http://blog.hypriot.com/downloads/) lets you run Docker on the pi.

### Registry

We will build the registry for ARM with `hypriot/rpi-golang` as the base image.

### Web server

We'll use the web server setup from the [official documentation](https://docs.docker.com/registry/apache/), but alter it slightly to work on pi. This will let us restrict pushing of images to an admin user.

# Setting it up

## Preparations

Before you begin, ensure that you have the latest versions of [Ansible](http://www.ansible.com) and [Docker](http://www.docker.com) on your local machine.

## OS installation

*Time required: 5 min*

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

## Pi setup

*Time required: 4 min*

The initial setup assumes that you already have an SSH keypair at `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`.

    # Add the Raspberry Pi IP to the file `hosts`
    # Replace 192.168.0.123 with the IP of your Raspberry Pi.
    echo "[pi]
    192.168.0.123" > hosts

    # Run the playbook
    # The password is `hypriot`
    ansible-playbook initial_setup.yml --user root --ask-pass


## Build Dockerimages

*Time required: 53 min*

    # Run playbook
    # Registry role takes 31 min
    # Apache role takes 22 min

    ansible-playbook build.yml

## Configure

*Time required: 10 min*

This is a variation on the script from the [official site](https://docs.docker.com/registry/apache/). You will need Docker locally for one of the tasks.

    # Ensure that you have put your certificate and key files
    # in ~/.ssl/registry/
    # Update the variables ssl_certificate and ssl_key in configure.yml
    # if your keys are elsewhere and/or have other names.

    # Run the playbook (replace domain name with yours)
    ansible-playbook configure.yml -e "registry_fqdn=yourregistrydomain.com"

    # Ensure your DNS record is updated with the IP of Raspberry Pi.
    # Then check (you may need to flush your DNS cache first)
    curl https://myregistrydomain.com

## Login

From your local machine, you can now start interacting with the registry.

    # Log in as admin
    docker login myregistrydomain.com
    Username: admin
    Password:
    Login Succeeded

    # Push an image
    docker pull busybox
    docker tag busybox myregistrydomain.com/library/busybox
    docker push myregistrydomain.com/library/busybox
    The push refers to a repository [myregistrydomain.com/library/busybox]
    5f70bf18a086: Pushed
    1834950e52ce: Pushed
    latest: digest: sha256:6757d4b17cd75742fc3b1fc1a8d02b45b637f2ac913ee9669f5c2aed0c9b26ba size: 711

    # Log in as the user
    docker login myregistrydomain.com
    Username: user
    Password:
    Email:
    Login Succeeded

    # Pull image
    docker pull myregistrydomain.com/library/busybox
    Using default tag: latest
    latest: Pulling from library/busybox

    385e281300cc: Already exists
    a3ed95caeb02: Already exists
    Digest: sha256:4a731fb46adc5cefe3ae374a8b6020fc1b6ad667a279647766e9a3cd89f6fa92
    Status: Image is up to date for myregistrydomain.com/library/busybox:latest

    # Try pushing as user (won't work!)
    docker push myregistrydomain.com/library/busybox
    The push refers to a repository [myregistrydomain.com/library/busybox]
    5f70bf18a086: Layer already exists
    1834950e52ce: Layer already exists
    unauthorized: authentication required

# Limitations

* Only tested on Raspberry Pi 3.
* Building takes Forever[tm] - plenty of room for slimming down images and shortening build time
