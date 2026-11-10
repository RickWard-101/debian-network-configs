# Debian APT & Network Troubleshooting Guide

## Issue 1: APT cdrom:// Repository Error
Symptom -> When running apt update, APT returns errors similar to:

Error:	2 cdrom://[Debian GNU/Linux ...] trixie Release
Error: 	O repository 'cdrom://...' not archive Release

The cause of this is because during installation from an ISO/media, Debian adds the installation disc as an
active repository in /etc/apt/sources.list. When the media is unmounted or disconnected,
apt fails to read package lists from the CD-ROM. Additionally, the online http:// mirrors are disabled by default (commented out with #).

__Solution -> Open:__

 	vim /etc/apt/sources.list

Comment out the cdrom: line by placing a # at the beginning.
Remove the # from the http://deb.debian.org lines.
*Example Configuration:*
```
	# deb cdrom:[Debian GNU/Linux 13.6.0 _Trixie_...
	deb http://deb.debian.org/debian trixie main contrib non-free-firmware
	#deb-src http://deb.debian.org/debian trixie main contrib non-free-firmware
	
	deb http://deb.debian.org/debian-security/ trixie-security main contrib non-free-firmware
	#deb-src http://deb.debian.org/debian-security/ trixie-security main contrib non-free-firmware
```

Save the file. Repository Components Reference	
main: 100% free and open-source software (DFSG-compliant).
contrib: Free software that depends on non-free software.
non-free-firmware: Closed-source drivers required for hardware initialization (Wi-Fi, GPUs). Enabled by default in modern Debian releases.
non-free: Proprietary, closed-source software (NVIDIA drivers, unrar, proprietary codecs).
To enable full non-free packages, append non-free to the end of the active repository lines in /etc/apt/sources.list.

## Issue 2: Temporary Failure Resolving Domain
Symptom -> When running apt update, APT returns warnings/errors:


Warning: Fail to obtain http://deb.debian.org/debian/... fail temporary to resolve 'deb.debian.org'
The cause of this is because the virtual machine lacks functional DNS server settings or its network interface is disconnected, preventing the system from resolving domain names to IP addresses.
*Diagnostic Steps* -> Test raw IP connectivity:

        ping -c 3 1.1.1.1

If this fails: The network adapter/interface is down or has no gateway route.
Test DNS resolution:

        ping -c 3 deb.debian.org

If step 1 worked but step 2 failed: Internet connectivity is present, but DNS server configuration is missing.
__Solution (Fixing DNS) -> Open__
	
	vim /etc/resolv.conf

Add public DNS servers to the top of the file:

        nameserver 1.1.1.1
        nameserver 8.8.8.8

Save and test resolution again:

        ping -c 3 deb.debian.org
        apt update

THIS WILL NOT SAVE AFTER BOOT BECAUSE /etc/resolv.conf WILL BE OVERWRITEN BY DHCP SERVER / STATIC 

