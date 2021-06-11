# esxi-vm-deploy
Offering a golang program to automate vm installation on a esxi host.
The program will be asking for some informations, templating a .vmk file, creating a virtual disk and installing operating system via embedded BOOTP python server.
Before starting the automatic deployment of virtual machines you will need to tune up your system in order to be able to run program.
To run this program you will need:
* go > 1.13
* ansible >= 2.7

## Download&Run!
You can download and run precompiled binaries by clicking [here](https://github.com/lucabodd/esxi-vm-deploy/releases/tag/1.2.7) and selecting your running platform

## Installation
If precompiled binaries are not working for you, you can read de guide below for the manual installation.

### Automatic System Setup
If you are running ***Debian***, in order to setup the system and be able to running golang program you can run the script below (needs sudo privilege on host) for automatic setup or you can follow the step-by-step guide in "Manual Setup" section
```
curl https://raw.githubusercontent.com/lucabodd/esxi-vm-deploy/master/setup/ESXi-vm-deploy-install.sh | bash
```

### Manual System Setup
If automatic system setup fails, or you want to setup the system manually, you can setup the system following the steps below.
In order to setup the system and being able to running golang program you will need to follow the following steps:
install ansible via package manager (Debian):
```
sudo apt-get install ansible
```
Install golang from sources
```
wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
tar -xvf go1.14.4.linux-amd64.tar.gz
sudo mv go /usr/local
rm -rf go
```
Install esxi-vm-deploy
```
go get github.com/lucabodd/esxi-vm-deploy
go install github.com/lucabodd/esxi-vm-deploy
```
In some cases you might need to export this environment vars
```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

## Update
As of today updates will be released on github.com, in order to update the program you can run the following command
```
go get -u github.com/lucabodd/esxi-vm-deploy
```

## Usage
Usage is described running the program with the help flag.
Before running the program, please consider the following checklist:
* "helper" host and "vm-ip" must be in the same target subnet (unless you are able to broadcast UDP packets :67,:68 outside the helper's server)
* one helper can deploy one vm per time, if you want to deploy more than one VM at once, you'll need more helpers (or wait :'))

```
Usage: esxi-vm-deploy [OPTIONS]
  -esxi string
        ESXi Hypervisor
  -help
        prints this help message
  -helper string
        BOOTP server address, specified host will provide configurations to booting (PXE) virtual machine
  -verbose
        enable verbose mode
  -version
        Display current version of script
  -vm-cpu string
        Specify RAM size (default "2")
  -vm-disk-size string
        Virtual machine Disk size (default "50")
  -vm-ip string
        Virtual machine IP address
  -vm-name string
        Specify virtual machine name
  -vm-os string
        Specify virtual machine OS available: debian9-64, debian10-64 (default "debian10-64")
  -vm-ram string
        Specify RAM size (default "16")
One ore more required flag has not been prodided.
Note that using less flag than required could lead program into errors
Omit flags only if you are aware of what are you doin'
[EXAMPLES]
- Creation of machine with custom hardware
esxi-vm-deploy --esxi [esxi host defined in ssh config] --helper [unix host defined in ssh config]  --vm-ip [ip of new machine] --vm-name [name of new machine] --vm-ram 8  --vm
-disk-size 16 --vm-cpu 4
- Creation of machine with default values 3 CPU 50GB Disk 16GB RAM
esxi-vm-deploy --esxi [esxi host defined in ssh config] --helper [unix host defined in ssh config]  --vm-ip [ip of new machine] --vm-name [name of new machine]
```

## Maintainance
To maintain this tool you need to periodically update debian netboot images in order to allow the the kernel version of the installer to match the version available in the debian archives.
To do this you need to save the files ```preseed.cfg``` and ```preseed_lvm.cfg``` located for example in ```esxi-vm-deploy/roles/vm-deploy/files/linux/pybootd/images/debian10-64 ``` in a temporary directory.
download the new installer version from debian repos.
```
wget --recursive --no-parent -nH --cut-dirs=8 --reject html,tmp http://ftp.nl.debian.org/debian/dists/stretch/main/installer-amd64/current/images/netboot/debian-installer/
```

once downloaded add the  ```preseed.cfg``` and ```preseed_lvm.cfg``` files inside the ```/debian-installer/amd64/``` directory.
created the directory replace the new debian installed dir in ```esxi-vm-deploy/roles/vm-deploy/files/linux/pybootd/images/debian10-64 ```
