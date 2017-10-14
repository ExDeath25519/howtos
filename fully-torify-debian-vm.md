# Route all the TCP traffic to TOR in a Debian GNU/Linux VirtualBox VM

1. According to wireshark, it's okay with IRC, web browsing and git. **USE AT
   YOUR OWN RISK** - i'm not an expert - for IRC don't forget to disable all CTCP
   queries and DCCs.
2. This blocks all UDP queries, excepted DNS ones, that are encapsulated in TCP
   packets, so your DNS queries go through Tor.
3. <u>**You shouldn't do this.**</u> [Use Whonix](https://www.whonix.org/) instead, or use
   a *middlebox* VM you've made yourself.
4. This quick HOWTO assumes Debian GNU/Linux is already installed, and no
   firewall are active. This should work for any Debian-based distro and can
   easily be adapted for others.
5. Your VM should be in NAT mode networking.

You'll need to be r00t for this whole HOWTO.

## Install Tor 

	apt install tor 

## Change Tor configuration 

Edit `/etc/tor/torrc` and add at the end of the file :

	VirtualAddrNetworkIPv4 10.192.0.0/10
	AutomapHostsOnResolve 1
	TransPort 127.0.0.1:9040
	DNSPort 127.0.0.1:5353

And then restart it : 

	service tor restart

## Reconfigure your network virtual adapter

Edit `/etc/network/interfaces` and instead of using DHCP, we will set a static
address, i chose 42 for obvious reasons: 

	auto enp0s3
	iface enp0s3 inet static
	address 10.0.2.42
	gateway 10.0.2.2


## Lock your `/etc/resolv.conf`

	echo "nameserver 127.0.0.1" > /etc/resolv.conf
	chattr +i /etc/resolv.conf

## Create iptables rules

Some explanations, because it may need some tweaking: 

* **10.0.2.42** is our VM ip address
* **107** is the uid of the `debian-tor` user, who runs the Tor service
* **9040** is the Tor port 
* **5353** is our 'DNS through TCP' port
* **YOU WILL NEED TWEAKING IF YOU HAVE SET MULTIPLE NETWORK ADAPTERS IN YOUR VM**

Create the `/etc/iptables.up.rules` file, with the following content: 

	*nat
	:PREROUTING ACCEPT [0:0]
	:OUTPUT ACCEPT [0:0]
	:POSTROUTING ACCEPT [0:0]
	-A PREROUTING -d 10.0.2.42 -j RETURN
	-A PREROUTING -p udp -m udp --dport 53 -j REDIRECT --to-ports 5353
	-A PREROUTING -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -j REDIRECT --to-ports 9040
	-A OUTPUT -p udp -m udp --dport 53 -j REDIRECT --to-ports 5353
	-A OUTPUT -o lo -j RETURN
	-A OUTPUT -m owner --uid-owner 107 -j RETURN
	-A OUTPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -j REDIRECT --to-ports 9040
	COMMIT
	*filter
	:INPUT DROP [0:0]
	:FORWARD DROP [0:0]
	:OUTPUT DROP [0:0]
	-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
	-A INPUT -p icmp -j ACCEPT
	-A INPUT -i lo -j ACCEPT
	-A INPUT -j LOG --log-prefix "IPv4 REJECT INPUT: "
	-A FORWARD -j LOG --log-prefix "IPv4 REJECT FORWARD: "
	-A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
	-A OUTPUT -o lo -j ACCEPT
	-A OUTPUT -d 127.0.0.1/32 -p udp -m udp --dport 5353 -j ACCEPT
	-A OUTPUT -d 127.0.0.1/32 -p tcp -m tcp --dport 9040 -j ACCEPT
	-A OUTPUT -m owner --uid-owner 107 -m conntrack --ctstate NEW -j ACCEPT
	-A OUTPUT -j LOG --log-prefix "IPv4 REJECT OUTPUT: "
	COMMIT

Once written:

	chmod 0600 /etc/iptables.up.rules

## Apply them at boot time 

They will be applied before our virtual ethernet adapter goes UP. 

Create the `/etc/network/if-pre-up.d/iptables` file, with the following content:

	#!/bin/sh
	/sbin/iptables-restore < /etc/iptables.up.rules

Again: 

	chmod 0700 /etc/network/if-pre-up.d/iptables

## Applying changes 

Well, just reboot. Alternatively you can run `/etc/network/if-pre-up.d/iptables`.

## Checking

	wget -qO- ifconfig.me/all
	curl ifconfig.me/all
