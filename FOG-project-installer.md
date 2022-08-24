# Installing a Deployment server

## Intro

In this document i'll explain how you can deploy a deployment server using:

- Ubiquity router (USG-Pro-4)
- Proxmox 7.0-11
- Debian 11
- Fog Project 1.59.159

## What is FOG

Fog is a deployment server with webinterface witch allows you to deploy and manage x86 based hardware.
I has a lot of features such as:
- creating full disk copies
- managing deployed images
- Product key managment
- Active directory support


## Creating a CT in Proxmox

Creating a CT in Proxmox is pretty straight forward.
However, there is a small thing that you need to check before installing FOG.
FOG uses NFS with does not work out of the box in proxmox.
To enable it you need to create a preveliged container.
You can select this option in the bottom left corner in the "add container" menu.

After you've create the container you need to go to "Options" and select "NFS" and "Nesting" under the "Features" item.

## Installing FOG

First update your CT with:

`apt-get update; apt-get full-upgrade`

After that you need to install `Git` for downloading the repo:

`apt-get istall git`

Finaly you can install FOG with 1 command:

`git clone https://github.com/fogproject/fogproject.git fogproject-dev-branch; cd fogproject-dev-branch; git checkout dev-branch; cd bin; ./installfog.sh`


### Default credentials

If you open the webinterface for the first time the default username is `fog` and the password is `password`.


## Adding the CT ipadress as network boot option in your Ubiquity router

After following the instructions in the install script you need to add the ipadress of your FOG server as a network boot entry in your Ubiquity router.

You can do this under `Settings->Network->YourNetworkName->DHCP->DCHP Service Management`

Enable `DHCP networkboot`

Under `Server` fill in the ip of your FOG server and under `filename` fill in `undionly.kkpxe`.

If you scroll down you can see `DHCP TFTP Server`.
In here you also fill in the ip of your FOG server.

Save everything and you're done.


## Creating an image

If you want to create a image you need to do a manual windows install on a machine first.
Install all the necesarcy drivers if you want to deploy identical devices.

After your done you need to shutdown Windows correctly because Windows does not "Shutdown" by default.
It will save some data so that you can startup the machine faster.

But this is not in our advantage, this means that the partitions cannot be read by FOG.

So Select `Shutdown` while holding the `shift` key.
After Windows shut's down boot with network boot and register the machine.
When done with that create a image task in the FOG dashboard.
Goto `Hosts->List all hosts->Task`.

Go back to your device and reboot into network boot.
The image creation procces will start automaticaly.

Wait untill cloning the partitions is done and shut down the machine.

## Deploying an image

Deploying a image is allmost the same as creating one.
Boot into network boot.
Register the machine.
And then roll out a deploy task via `Hosts->List all hosts->Task`.

### NOTE this procces will remove ALL content on the current disk inserted in the machine.


## done
You can now create and deploy images at lightning speed.
Deploying takes around 5 minutes per machine.
Have fun and if you have any questions or issues with this installation guide you can mail me via:
`mauricenorden@gmail.com`  
