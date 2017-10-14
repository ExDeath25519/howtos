# Rename you and your computer "from the bottom to the ceiling"

## Introduction

Why ?

I've leaked too many things under pseudonyms that crossed paths with the
real-life me, all these years. So i decided to make a strong split between the
real-life me and the new pseudonymous ExDeath25519 this time, and i started @
localhost indeed.

## What to do ? 

In my humble opinion, the best way to do this is a fresh, encrypted,
installation on another : 

* virtual computer (most convenient)
* computer (that sounds pretty cool, like airgapped computers)
* partition, this includes a live distro with some encrypted persistent
  storage (not very convenient)
	
as your pseudonymous system, while using your current installation as your real
life one, after purging/moving "pseudonymous" infos, and this HOWTO focus on
the latter. 

Following this HOWTO won't make you bulletproof - this is you and how you act
on the net and in real life that will *almost* do everything. This HOWTO is not
exhaustive, and assume you have used very personal names for "everything". 

## Change your login name and move $HOME data

**WARNING**: *because of this, many of your scripts, shortcuts, settings
(network-manager, virtualbox, blueman...) etc.  will be "missing".*

If you lack inspiration, just use *user*.

At first, in you current session get your id, and write it down:

	$ id

* Log off
* Press Ctrl-Alt-F1 to get to the tty console.
* Login as root
* Enter the following commands : 

	``` 
	pkill -U <yourid>
	usermod -l newlogin oldlogin -d /home/newlogin -m 
	groupmod -n newlogin oldlogin
	```
Oh, also change your account "realname" : 

	chfn -f "   " newlogin

Don't forget to check your `/etc/sudoers` too.

## Change your autoconnect name in your DM

It's kinda okay to use autoconnect IF you're using an encrypted drive asking
for a password at boot, and IF your display manager doesn't reconnect you
automatically if you disconnect.

I use lightdm, so i changed my `/etc/lightdm/lightdm.conf` to put my new login
name instead of the old.

## Change your hostname

You'll need to edit `/etc/hostname` and put your new hostname there. If you
lack inspiration, *localhost* is surely the most generic thing you can have.
There will need at least an occurence of `localhost`, anyway if not, bad stuff
may occur : 

	127.0.0.1	localhost.localdomain

We won't reboot now, because i guess your hard drive is encrypted, and you put
some meaningful name in it ?

## Renaming your LVM volume group

If encryption is used by LVM, the volume group name pop ups at boot time.

This one is risky if you don't do things properly. I could have made a shell
script using sed, but if you use a very generic name like 'linux', it would
break everything. And yes you can do it on your live system, no need for a
rescue disk.

The following instructions cover an almost fully encrypted disk under one VG,
with `/boot` unencrypted (a somewhat classical encrypted setup).

We'll start by renaming the LVM VG, again if you lack inspiration, your distro
name is your best option as a new name : 

* We need its name first

	```
	vgdisplay | grep "VG Name"
	```
* Now you have the name, actually change it :

	```
	lvm vgrename OLDVGNAME NEWVGNAME
	```

You will need to remplace all occurencies of `OLDVGNAME` by `NEWVGNAME` in these files : 

* /etc/fstab 
* /boot/grub/grub.cfg
* /etc/initramfs-tools/conf.d/resume

And then : 

	update-initramfs -u
	sync

Is a poor man's validation for the changes. 

Reboot now, don't worry it will take ages to shutdown, because the volume name
changed - if it's too long then force the reboot :'(

## Change your bluetooth adapter name

Nothing more to say here, excepted it shouldn't match your new hostname. I
haven't found a user-friendly way to change the mac address, and anyway it's
not supported by my bluetooth card. 

If you know, let me know.

## Spoof your mac addresses

Many networks store mac addresses, and not only for association and stuff ;)
Furthermore, an attacker using a sniffer like wireshark can get your network
device (like IntelCor\_33:44:55 or SamsungE\_AA:BB:CC). 

Note that boot-time randomizing mac addresses can be a PITA when used on a wifi
device, because you'll need to reassociate each time your mac changes.

Network-manager allows you easily to spoof MAC addresses : 

* Right-click on your network-manager appel 
* Click "Edit connections" 
* Choose your connection 
* Click on "Modify"
* You've a "cloned mac address" combo box. Choose "Random" or use a fixed
  one if you need it.

You're set. Or not.

## Stuff you should take care of 

There were more things here, but my plans changed meanwhile, and installing a 
pseudonymous system fix MANY things already.

###  ssh keys

Don't forget to add `-C ' '` to your `ssh-keygen` invocations, and yes you
should modify all `~/.authorized_keys`

### software repos

Probably you have set your distribution repo in your country. Change it, in case
you share your sources.list or whatever your distro is using for defining
software repos.

### logs

Logs are usually restricted to root. **Usually** is the keyword.  

### old nicks etc.

It's a super boring task, and it's the last. And it's where i can't really help
you. Your files, your game.

* Find all occurencies of your old nickmames in `$HOME`
	```
	grep -ri oldnick ~ 
	```
* Find all filenames that include old nicknames :
	```
	find ~ -name '*oldnickname*'
	```

I would extend the search to your whole `/`.

## And finally

Move your pseudonymous stuff to your pseudonymous installation. You should explore
TOR and VPNs next to even split the network traffic between you and your pseudonyms :D
