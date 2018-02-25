
## Table of Contents

* [Requirements](#requirements)
* [Installation](#installation)
  * [Install a minimal Debian](#install-a-minimal-debian)
    * [B.A.T.M.A.N. and fastd](#b.a.t.m.a.n.-and-fastd)
    * [add repo.universe-factory.net repository](#add-repo.universe-factory.net-repository)
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

pick a secure root password.
 create a regular user on the host, then limit Root from being able to log in via ssh
    
       apt-get update
       apt-get upgrade
    
 2. Stuff that is needed
   
    
## Install a minimal Debian

## B.A.T.M.A.N. and fastd

## add repo.universe-factory.net repository

## downgrade to batman 14

## This is needed for any version

## Networking

## DHCP and DNS

## DHCP radvd IPv6

## DHCP isc-dhcp-server IPv4 and IPv6

## DNS bind9

## VPN
