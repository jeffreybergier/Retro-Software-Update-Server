![Retro Software Update Server](images/001-Title.png)

# Retro-Software-Update-Server
A guide on setting up an Apple Software Update Server Mirror in case Apple 
ever decides to shut down these old update servers. So far software update
still works on Mac OS X 10.4 and above, but Apple already shut down servers
for 10.3, 10.2, 10.1, and 10.0 and those can't be recovered.

If you have an interest in retro Macs, this may be something you would like
to set up in case something bad were to happen.

## Approach

1. Set up Mac OS X Server 10.6
1. \[To-Do\] Configure the web server to host a mirror of Apple's catalog files
1. \[To-Do\] Configure the software update server to use this mirror
1. \[To-Do\] Enable and Download all of the updates
1. \[To-Do\] Configure your retro Mac to use this server

## System Requirements

I recommend using an Intel Mac running VMWare Fusion to run the Snow Leopard
Server. I tried using UTM/QEMU on my Apple Silicon powered Mac and I couldn't
get anything to run. And any prebuilt images were not Server version and so
they did not come with Apple's Software Update Server. Of course, the Server 
doesn't need to be a VM, so if you want to use an old Intel Mac for this, 
that would work as well.

Another option is to use a more modern OS. In 10.7 Lion, Apple switched to
offering a Server App which was purchased and downloaded from the App Store. 
The problem is that 10.7 cannot connect to the App Store, and this App is
no longer for sale. Its only available if you already purchased it. Also, there
is version 5 of the Server App that is easier to get but it does not have the
the software update server. So our options are limited unfortunately.

# Guide

## 1. Set up Mac OS X Server 10.6

Sorry, but I am not going to cover this as it basically involves piracy. But 
you need to do the following:

1. "Get" a copy of Mac OS X Server 10.6 and a serial number
1. Get a copy of VMWare Fusion. Its free now, but downloading it can be a pain in the butt.
1. Install Mac OS X Server in a VM and get it to the Desktop

Points for the VM:
1. Make the Virtual Drive large (120+GB): The software updates actually take a lot of space on the disk.
1. Make sure to use Bridged networking: Most VM products offer "Shared" or "Bridged" networking. Bridged is essential so the VM sits on your network like any other computer.
