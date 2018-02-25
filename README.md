
## Table of Contents

* [Requirements](#requirements)
* [Installation](#installation)
    * [B.A.T.M.A.N. and fastd](#b.a.t.m.a.n.-and-fastd)
    * [downgrade to batman 14](#downgrade-to-batman-14)
    * [This is needed for any version](#this-is-needed-for-any-version)
* [Networking](#networking)
* [DHCP and DNS](#dhcp-and-dns)
  * [DHCP radvd IPv6](#dhcp-radvd-ipv6)
  * [DHCP isc-dhcp-server IPv4 and IPv6](#dhcp-isc-dhcp-server-ipv4-and-ipv6)
  * [DNS bind9](#dns-bind9)
* [VPN](#vpn)

## Requirements
    min. 2 cores with good thread performance
    min. 512 MB RAM
    min. 10 GB HDD
    approx. 5 TB/Month traffic allowance is a good start.
    you have to be able to modify the kernel on your server, so OpenVZ or the likes won't work
    min. 100 Mbit/s connectivity is recommended
    recommended is also a VPN provider for tunneling the traffic out

## Installation
1. Install a minimal Debian jessie

       make sure to set the hostname and domainname:
       hostname $FREEMESH_HOSTNAME
       echo "127.0.0.1 $FREEMESH_DOMAIN_NAME $FREEMESH_HOSTNAME" >>/etc/hosts
       echo $FREEMESH_HOSTNAME > /etc/hostname

* pick a secure root password.
* create a regular user on the host, then limit Root from being able to log in via ssh
    
       apt-get update
       apt-get upgrade
    
 2. Stuff that is needed
 
        apt-get install screen htop iftop traceroute mtr-tiny mc speedtest-cli openvpn bash-completion nano ca-certificates haveged ntp unbound isc-dhcp-server radvd iptables-persistent apt-transport-https postfix bind9 sudo git apt-transport-https

## B.A.T.M.A.N. and fastd

add jessie-backports to your /etc/apt/sources.list, then

    echo "deb https://repo.universe-factory.net/debian/ sid main" >>/etc/apt/sources.list
    gpg --keyserver pgpkeys.mit.edu --recv-key  16EF3F64CB201D9C
    gpg -a --export 16EF3F64CB201D9C | apt-key add -

    apt-get update
    apt-get install batctl fastd bridge-utils

## downgrade to batman 14
    modinfo batman-adv
    apt-get install batman-adv-dkms
    dkms remove batman-adv/2013.4.0 --all
    dkms --force install batman-adv/2013.4.0
    modprobe batman-adv # (if the wrong version is loaded "rmmod batman-adv" and then repeat the dkms commands)
    dmesg or batctl -v # (check, to see, that you have the correct version loaded)
## This is needed for any version
add batman-adv to your /etc/modules to autoload it on boot and activate it:
   
     echo "batman-adv" >> /etc/modules
     modprobe batman-adv
     
create the directory for your fastd peers:
     
     mkdir -p /etc/fastd/vpn/peers
     
create the keys for fastd:

    fastd --generate-key > /root/fastd-keys.pub.sec
    
Backup the key pair `/root/fastd-keys.pub.sec` in a safe place. This will be needed both for the server and for the image builds.

create the .conf for fastd:

nano /etc/fastd/vpn/fastd.conf

    #log level warn;
    #hide ip addresses yes;
    #hide mac addresses yes;
    bind any:10006 interface "eth0";
    method "salsa2012+umac";  	#
    mtu 1406;			 # 1280 bytes for the client and 24 bytes for the batman unicast header
    include "secret.conf";  	# include the secret.conf with your private secret key from the /root/fastd-keys.pub.sec 
    include peers from "peers";     # your peerlist with the other gateways
    on verify "true";		# this allows any unknown node to connect to your fastd
    on up "
      /sbin/ifup bat0
      /bin/ip link set dev tap0 address 00:00:ff:f1:01:[GWnumber]
      /bin/ip link set dev tap0 up
    ";
    
insert your private secret key from the /root/fastd-keys.pub.sec in the secret.conf:

    secret "$your_private_secret_key_from_the_/root/fastd-keys.pub.sec";
    
For the gateway to be able to connect to the other gateways and nodes known in the network, you need to get a bunch of files with the public keys for these. For Freifunk Nord for example, this looks like this: 

    git clone https://github.com/Freifunk-Nord/nord-gw-peers /etc/fastd/vpn/peers
    
reload fastd without quitting: 
  
    killall -HUP fastd
    
if it says "no process found", its because you just installed it and it doesn't run, so just start it: 
  
    service fastd start
    
Regularly update the peers via cron.

    #!/bin/sh

    # hop into correct directory to avoid cron pwd sucks
    cd $(dirname $0)

    # function to get the current sha-1
    getCurrentVersion() {
    git log --format=format:%H -1
    }

    # get sha-1 before pull
    revision_current=$(getCurrentVersion)

    git pull -q

    # get sha-1 after pull
    revision_new=$(getCurrentVersion)

    # if sha-1 changed, make fastd reload the keys
    if [ "$revision_current" != "$revision_new" ]
    then
    kill -HUP $(pidof fastd)
    fi

make the script executable

    chmod +x /etc/fastd/reloadPeers.sh
    
and add these lines in cron with

    sudo crontab -e
    
    # Regularly update the fastd peers
    */5 * * * * /etc/fastd/reloadPeers.sh


## Networking

    Edit the file /etc/sysctl.conf and find the section

    # Uncomment the next line to enable packet forwarding for IPv4
    net.ipv4.ip_forward=1

    # Uncomment the next line to enable packet forwarding for IPv6
    #  Enabling this option disables Stateless Address Autoconfiguration
    #  based on Router Advertisements for this host
    net.ipv6.conf.all.forwarding=1

The two forwarding rules will be commented out, so you need to remove the # in front of them

Edit the file /etc/network/interfaces:

      # This file describes the network interfaces available on your system
      # and how to activate them. For more information, see interfaces(5).

      source /etc/network/interfaces.d/*

      # The loopback network interface
      auto lo
      iface lo inet loopback

      # The primary network interface
      auto eth0
      allow-hotplug eth0

      iface eth0 inet dhcp

      # Batman
          auto br-fffl
          allow-hotplug bat0

      iface eth0 inet6 static
          address [your public ipv6]  # don't change
          netmask [your public ipv6 netmask]  # don't change
          gateway [your public ipv6 gateway]  # don't change

      #  HOSTER: Everything above this line is the config from your hoster
      # <----------------------------------------------------------------------
      #  FREEMESH: Everything after this line is the config for your community

      # Batman
          up /sbin/brctl addbr br-fffl

      iface br-fffl inet static
          address 10.129.1.[GWnumber]
          netmask 255.255.0.0
          bridge-ports none

      iface br-fffl inet6 static
          address fddf:bf7:10:1:1::[GWnumber]
          netmask 64

      # batman interface
      iface bat0 inet6 manual
         pre-up /usr/sbin/batctl if add tap0
         up /bin/ip link set dev bat0 up
         post-up /sbin/brctl addif br-fffl bat0
         post-up /usr/sbin/batctl it 10000
         post-up /usr/sbin/batctl gw_mode server
         post-up /sbin/ip rule add from all fwmark 0x1 table 42
         post-up /sbin/ip route add 10.129.0.0/16 dev br-fffl table 42
         post-up /sbin/ip -6 rule add from all fwmark 0x1 table 42
         post-up /sbin/ip -6 route add fddf:bf7:10:1:1::[GWnumber]/64 dev br-fffl table 42
         pre-down /sbin/brctl delif br-fffl bat0 || true
         down /bin/ip link set dev bat0 down

         # this is for adding the gateway to the map later, not necessarily necessary.
         # post-up start-stop-daemon -b --start --exec /usr/local/sbin/alfred -- -i br-$FREEMESH_TLD -b bat0 -m
         # post-up start-stop-daemon -b --start --exec /usr/local/sbin/batadv-vis -- -i bat0 -s

      source /etc/network/interfaces.d/*


restart the network, so debian apply the changes in the /etc/network/interfaces:

         /etc/init.d/networking restart
         
give the new IP4 and IPv6 to the bridge:

    ifconfig br-fffl 10.129.1.[GWnumber]
    ip a a fddf:bf7:10:1:1::[GWnumber]

## DHCP and DNS

DHCP radvd IPv6

The file /etc/radvd.conf has to be edited.

    interface br-fffl {
      AdvSendAdvert on;
      #
      # Please uncomment if you don't want an IPv6 default route to be broadcasted.
      #
      # AdvDefaultLifetime 0;
      IgnoreIfMissing on;
      AdvManagedFlag off;
      AdvOtherConfigFlag on;
      MaxRtrAdvInterval 200;

      prefix fddf:bf7:10:1:1::/64 {
     AdvOnLink on;
     AdvAutonomous on;
     AdvRouterAddr on;
     AdvPreferredLifetime 14400;
     AdvValidLifetime 86400;
      };

      RDNSS fddf:bf7:10:1:1::[GWnumber] {
      };

      route fc00::/7
      {
     AdvRouteLifetime 1200;
      };
    };
    
   You can restart it now. 

       service radvd restart

## DHCP radvd IPv6

## DHCP isc-dhcp-server IPv4 and IPv6

## DNS bind9

## VPN
