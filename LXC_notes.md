# LXC Containers

## Install dependencies, EPEL repo needed

    yum install debootstrap perl libvirt

## Install LXC

    yum install lxc lxc-templates
    yum -y install lxc lxc-templates libcap-devel libcgroup busybox wget bridge-utils lxc-extra

## Enable LXC

    systemctl status lxc.service
    systemctl start lxc.service
    systemctl start libvirtd 
    systemctl status lxc.service

## Check kernel virtualization status

    lxc-checkconfig

### LXC comes with ready-made templates for easy installation of containers, and you can list down the available templates using the following command.

    ls /usr/share/lxc/templates/

## List available templates

    ls -alh /usr/share/lxc/templates/

## Create container using alpine template

    lxc-create -n <container_name> -t lxc-alpine

## List info about container

    lxc-ls
    lxc-info <container_name>
    
### To log into the container (centos_lxc), either use the temporary root password stored in the following location. In our case, “Root-centos_lxc-KRzJLy” is the root password of the centos_lxc.

    cat /var/lib/lxc/centos_lxc/tmp_root_pass
    >>>> Root-centos_lxc-KRzJLy

### or Reset the root password using the following command.

    chroot /var/lib/lxc/centos_lxc/rootfs passwd

    PS: You are yet to start the containers.

## Start container and run as daemon

    lxc-start -n mycontainer -d

## Connect to the console

    lxc-console -n centos_lxc -t 0
    Note: I use “-t” with “0” to connect the container with tty0, just because tty1 was not responding to me.

## List only running container

    lxc-ls --active
    
## Stop container

    lxc-stop -n mycontaier

### In order to create a LXC container based on an Ubuntu template on RHEL7, enter /usr/sbin/ directory and create the following debootstrap symlink

    cd /usr/sbin
    ln -s debootstrap qemu-debootstrap

#### Edit qemu-debootstrap file with Vi editor and replace the following two MIRROR lines as follows:

    DEF_MIRROR=”http://mirrors.kernel.org/ubuntu”
    DEF_HTTPS_MIRROR=”https://mirrors.kernel.org/ubuntu”

#### For reference, see the following content and place the above two lines as stated:

    ....
    MAKE_TARBALL=""
    EXTRACTOR_OVERRIDE=""
    UNPACK_TARBALL=""
    ADDITIONAL=""
    EXCLUDE=""
    VERBOSE=""
    CERTIFICATE=""
    CHECKCERTIF=""
    PRIVATEKEY=""
    DEF_MIRROR=”http://mirrors.kernel.org/ubuntu”
    DEF_HTTPS_MIRROR=”https://mirrors.kernel.org/ubuntu”

#### Or Run the following command to replace Debian mirror to ubuntu mirror.

    sed -i 's/DEF_HTTPS_MIRROR="https:\/\/mirrors.kernel.org\/debian"/DEF_HTTPS_MIRROR="https:\/\/mirrors.kernel.org\/ubuntu"/g' /usr/sbin/debootstrap

## Finally create a new LXC container based on Ubuntu template issuing the same lxc-create command.

#### Once the process of generating the Ubuntu container finishes a message will display your container default login credentials as illustrated on the below screenshot.

    lxc-create -n myubuntu -t ubuntu

## In order to create a specific container based on local template use the following syntax:

    lxc-create -n container_name -t container_template -- -r distro_release -a distro_architercture 

#### Here is an excerpt of creating a debian wheezy container with an amd64 system architecture.

    lxc-create -n mywheezy -t debian -- -r wheezy -a amd64

### For instance, specific containers for different distro releases and architectures can be also created from a generic template which will be downloaded from LXC repositories as illustrated in the below example.

    lxc-create -n mycentos6 -t download -- -d centos -r 6 -a i386

#### Here is the list of lxc-create command line switches:

    -n = name 
    -t = template
    -d = distibution
    -a = arch
    -r = release

## Delete LXC container

    lxc-destroy -n mywheezy

## Clone container

    lxc-clone mydeb mydeb-clone

## Get info

    lxc-info -n centos_lxc

## Taking a Snapshot

#### LXC also has another feature called snapshot, use the following commands.

##### Note: You must stop a container before taking a snapshot.

    lxc-stop -n centos_lxc_clone
    lxc-snapshot -n centos_lxc_clone

#### To know where the snapshot is being saved, run the following command.

    lxc-snapshot -L -n centos_lxc_clone

## Restoring Snapshot

#### To restore a container from the snapshot, use the following command.

    lxc-snapshot -r snap0 -n centos_lxc_clone

### All created containers reside in /var/lib/lxc/ directory. If for some reason you need to manually adjust container settings you must edit the config file from each container directory.

    ls /var/lib/lxc

--

# LXD

## Installing LXD
    
    Ubuntu:
    
    sudo apt-get install lxd lxd-client

## Start LXD

    sudo service lxd start
    sudo lxd init

## First steps

    sudo lxc-list #It will gernate some certificates at first launch
    
## Create a LXC container

    lxc launch ubuntu:16.04 ubuntu
    lxc launc images:alpine/3.5 alpine

## List containers

    lxc list

## Exec commands

    lxc exec ubuntu -- /bin/bash
    lxc exec alpine -- ash
    
## Troubleshoot LXD installations

### Setup bridge before installing and check the name

    ifconfig
    sudo dpkg-reconfigure -p medium lxd
    
## Launching containers

## List images
    
    lxc image list

## Download image

    lxc image copy images:alpine/3.5 local: --alias alp
    lxc image list

## Launch container
    
    lxc launch alp web
    lcx list 
    Where: alp is the image name and web is the container name
    
## Exec on container

    lxc exec web -- apk update && apk add nginx
    
## Acces the console

    lxc exec web -- ash
    
## Edit files

    lxc file edit web/etc/nginx/conf.d/defualt.conf

## Copy files to container

    lxc file push index.html web/var/www/index.html
    
## Add nginx to start

    lxc exec web -- rc-update add nginx default
    
    Note: This is for Alpine. It can be done with other init scripts or systemctl enable nginx
    
    lxc-exec web -- /etc/init.d nginx start

## Take snapshot

    lxc snapshot web initialconfig
    lxc list
    
## Get info

    lxc info web

## Restore from snapshot

    lxc restore web initialconfig
    
## Create container from snapshot

    lxc copy web/initialconfig web2
    lxc list

##  Start container

    lxc start web2
    lxc list
    
## Delete container snapshot

    lxc delete web/initialconfig
    lxc list
    
## Delete container
    
    lxc stop web
    lxc delete web
    lxc list
    
## Delete images

    lxc image list
    lxc image delete alp

## Images

## List remote image servers

   lxc remote list
   
## Add remote LXD server

    On remote LXD image remote server (remoteserver)
    
    lxc config set core.https_address "[::]:8443"
    lxc config set core.trust_password supersecretpassword
    
    On my local LXD host
    
    lxc remote add remoteserver 172.31.23.109:8443 --password=supersecretpassword
    lxc list                   #List my local LXD host
    lxc list remoteserver:     #List my remote LXD server  
    
## List remote LXD server images
    
    lxc image list
    lxc image list remoteserver:   

## Publish container or snapshots to images

    lxc publish web/initialconfig --alias mywebImage
    lxc image list
    
## Launch container from created image

    lxc launch mywebImage webcontainer_new
    lxc list

## Create custom image

   mkdir webimage
   lxc image export mywebImage webimage/  #Creates the tar image file
   cd webimage
   tar xvfz tar_file.tar.gz #Importan to check the metadata.yaml and templates files, it has a rootfs directory
   
## Image from scratch

   images.linuxcontainers.org for the list of available images
   We can use debootstrap to create an initial image or use an image from the url
   Edit the metadata.yaml to customize
   Create tar file
   
   lxc image import tar_new_created_image.tar.gz
   lxc image list
   
## Images local path

    ls /var/lib/lxd/containers
   
## ZFS on LXD

### Installing

   sudo apt-get update && sudo apt-get install zfsutils-linux

### Setup a block device to use with ZFS

   There are several ways to do it, onde done check with:
   fidsk -l
   lsblk
   Use the device listed on those commands, the ZFS storage backend must be configured using:
   sudo lxd init

### Check ZFS pool

    sudo zpool list
    
## LXC/LXD Networking

    lxc exec mycontainer -- ifconfig

### Port forwarding

    user@linuxacademy:~$ sysctl net.ipv4.ip_forward
    net.ipv4.ip_forward = 1

    Note that if IP forwarding is not enabled on a host -- that is to say, the previous command returns a 0 -- it will require IP forwarding to be enabled.  This can be accomplished with these commands:

    echo 1 > /proc/sys/net/ipv4/ip_forward
    sysctl -p /etc/sysctl.conf

    Then the network will need to be restarted.  On our Ubuntu 16 image on Linux Academy's Cloud Servers, the command is:

    /etc/init.d/procps restart

    sudo iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 80 -j DNAT --to 10.24.69.17:80 

## Advanced Container Networking

### Two hosts Alpha and Bravo with GRE Tunnel between the host bridges (lxdbr0)

    Install LXD and start it and run:
    lxd init
    
    #Differentiate on each host by declaring variables
    export $LOCAL_IP=x.x.x.x
    export $REMOTE_IP=y.y.y.y
    
    #On each host
    sudo apt-get install lxd zfsutils-linux bridge-utils
    
    #Only on Alpha, Bravo will get the GRE tunnel
    sudo lxd init
    lxc list
    lxc launch images:alpine/3.5 mycontainer3
    
### Build the GRE Tunnel
    
    #Host Alpha
    sudo ip link add container_gre type gretap remote $REMOTE_IP local $LOCAL_IP ttl 255   
    sudo brctl addif lxdbr0 container_gre
    sudo ip link set container_gre up
    
    #Host Bravo
    sudo brctl addbr multibr0
    sudo ifconfig multibr0 up
    ifconfig
    sudo ip link add container_gre type gretap remote $REMOTE_IP local $LOCAL_IP ttl 255
    sudo brctl addif multibr0 container_gre
    sudo ip link set container_gre up
    sudo lxd init #Set the multibr0 bridge on Bravo, answer no to configure a new bridge, answer yes to use an existing bridge 
   
    #Host Alpha
    lxc remote add bravo $REMOTE_IP:8443 --password=secret
    lxc list
    lxc list bravo:
    lxc launch images:alpine/3.5 bravo:mycontainer4
    lxc list bravo:
    
    #After all this configuration both host containers will be able to communicate
    lxc exec mycontainer3 -- ash
    lxc exec bravo:mycontainer4 -- ash
    
### Move container between hosts, only on bare metal

    lxc stop mycontainer3
    lxc move mycontainer3 bravo:mycontainer3_bravo #It will preserve the same IP as both are on the same GRE tunnel
    lxc start bravo:mycontainer3_bravo
    lxc list bravo:
    


    
    
