# Installing a Deployment server
 
## Intro
 
In this document i'll explain how you can deploy a deployment server using:
 
- Ubiquity router (USG-Pro-4)
- Proxmox 7.0-11 [link](https://www.proxmox.com/en/://)
- Debian 11 [link](https://www.debian.org/)
- Fog Project 1.59.159 [link](https://fogproject.org/)
 
## What is fogproject
 
The fog project is a deployment server with web interface which allows you to deploy and manage x86 based hardware.
 
The creators have called it FOG because Fog computing is a decentralized computing infrastructure in which data, compute, storage and applications are located somewhere between the data source and the cloud.
The FOG project let's you manage all of that compute
 
I has a lot of features such as:
- creating full disk copies
- managing deployed images
- Product key management
- Active directory support
 
But we will only roll out 1 image in this example.
Don't worry, the setup guide is complete so if you want you can thinker with the other features down the line without any big issues.
 
They also have a great wiki where they document [everything](https://wiki.fogproject.org)
 
## What is Proxmox
 
Proxmox is a type 1 hypervisor based on debian.
You can manage it in multiple ways:
- Via the web interface
- Via the CLI
- Via the API
 
It has 2 ways of virtualization:
 
- LXC (Container/CT)
- KVM ("full vm")
 
LXC is the most efficient way to virtualize since it doesn't emulate a kernel.
Instead it is more of an isolated container witch gets its resources via the host operating system kernel.
In this example we'll use LXC because you can deploy it way faster than a KVM virtual machine.
 
## The specific network setup:
 
Before we get started i want to explain the current network setup in my lab.
 
I have a SFF pc running Proxmox.
It get's it internet via an unmanaged switch witch is connected to a USG-Pro-4 router.
This router has its own isolated subnet which has its own DHCP server.
 
`Proxmox->FOG container->Switch->router->`
 
`192.168.2.201->192.168.2.222-> * -> 192.168.2.254`
 
So in this example we'll use the DCHP server from the router.
But you can install a DHCP server via FOG.
Note that if you have multiple DHCP servers running within the same subnet of vlan.
 
They will interfere with each other.
A client will just send a ARP request and the DHCP server that is the fastest will handle the request.
 
## Creating a CT in Proxmox
 
Creating a CT in Proxmox is pretty straight forward.
However, there is a small thing that you need to check before installing FOG.
FOG uses NFS with does not work out of the box in proxmox.
To enable it you need to create a privileged container.
You can select this option in the bottom left corner in the "add container" menu.
 
And you need to give the container a static ip.
Otherwise the entrypoint won't be correct in our router if the container needs a reboot.
 
After you've created the container you need to go to "Options" and select "NFS" and "Nesting" under the "Features" item.
 
## Installing FOG
 
First update your CT with:
 
`apt-get update; apt-get full-upgrade`
 
After that you need to install `Git` for downloading the repo:
 
`apt-get istall git`
 
Finaly you can install FOG with 1 command:
 
`git clone https://github.com/fogproject/fogproject.git fogproject-dev-branch; cd fogproject-dev-branch; git checkout dev-branch; cd bin; ./installfog.sh`
 
 
### Default credentials
 
If you open the web interface for the first time the default username is `fog` and the password is `password`.
 
 
## Adding the CT ip address as network boot option in your Ubiquity router
 
After following the instructions in the install script you need to add the ip address of your FOG server as a network boot entry in your Ubiquity router.
 
You can do this under `Settings->Network->YourNetworkName->DHCP->DCHP Service Management`
 
Enable `DHCP network boot`
 
Under `Server` fill in the ip of your FOG server and under `filename` fill in `undionly.kkpxe`.
 
If you scroll down you can see `DHCP TFTP Server`.
In here you also fill in the ip of your FOG server.
 
Save everything and you're done.
 
 
## Creating an image
 
If you want to create an image you need to do a manual windows install on a machine first.
Install all the necessary drivers if you want to deploy identical devices.
 
After your done you need to shutdown Windows correctly because Windows does not "Shutdown" by default.
It will save some data so that you can startup the machine faster.
 
But this is not in our advantage, this means that the partitions cannot be read by FOG.
 
So Select `Shutdown` while holding the `shift` key.
After Windows shut's down, boot with network boot and register the machine.
When done with that, create an image task in the FOG dashboard.
Goto `Hosts->List all hosts->Task`.
 
Go back to your device and reboot into network boot.
The image creation process will start automatically.
 
Wait until cloning the partitions is done and shut down the machine.
 
## Deploying an image
 
Deploying a image is almost the same as creating one.
Boot into network boot.
Register the machine.
And then roll out a deploy task via `Hosts->List all hosts->Task`.
 
### NOTE this process will remove ALL content on the current disk inserted in the machine.
 
 
## done
You can now create and deploy images at lightning speed.
Deploying takes around 5 minutes per machine.
Have fun and if you have any questions or issues with this installation guide you contact me via the comments below this article.
