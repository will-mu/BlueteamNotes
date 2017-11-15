(Ubuntu) DNS Notes
======

##### DMZ (Demilitarized Zone)
- Outward facing network inbetween trusted internal network (LAN) and untrusted external network such as the internet
- Segregated from personal files
- Typically containing devices accessible to internet traffic, such as Web and DNS servers

##### DNS (Domain Name Server)
- Server maintaining a directory of domain names and translate them to IP addresses
	- www.google.com -> 201.23.52.1
	- 201.23.52.1 -> www.google.com
- Internet Service Providers view DNS servers to translate a web address you type into an IP address
- DNS Zone: a set of DNS records for a single domain 
- DNS Record : single entry of instructions on handling requests based on types for a zone
	- _A Record_ : Specifies IPv4 Address for a given host 
		- www.google.com -> 201.23.52.1
	- _AAAA Record_ (quad-A record): specifies IPv6 address for given host
		- www.google.com -> 2001:db8::7348
	- _CNAME Record_: specifies a domain name that has to be queried in order to resolve the original DNS query 
		- also used to create aliases of domain names
		- same server can be accessesed through documents.example.com and docs.example.com because of CNAME
	- _MX Record_: specifies a mail exchange server for a DNS domain name
		- the information is used by Simple Mail Transfer Protocol (SMTP) to route emails to proper hosts
	- _PTR Record_: (reverse of A and AAAA DNS Records) used to look up domain names based on IP addresses
	
### Installation and set up
- [Install Ubuntu](https://www.ubuntu.com/download/server)
- Change network options to `Host-Only, DMZ`
- Type in `ip link show` into terminal and you should see: lo, enp0s3
	- `ip link show` : shows information for all interfaces 
	- lo : loopback
	- enp0s3 : virtual network driver
- Add another interface
	- type `sudo nano /etc/network/interfaces` to edit the network interfaces configuration file with console based text editor nano
	- add 
	```
	auto enp0s3
	iface enp0s3 inet static
	address 172.20.240.23
	netmask 255.255.255.0
	gateway 172.20.240.254
	dns-nameservers 8.8.8.8
	```
	- type `sudo service networking restart`
	- verify netowrk interfaces with `ifconfig`
	- verify connectivity with `ping 8.8.8.8` 
		- `8.8.8.8` is google's DNS server

### Configure DNS Server
- [Helpful link for configuration process](https://www.ostechnix.com/install-and-configure-dns-server-ubuntu-16-04-lts/)
- [Helpful link for understanding bind](http://www.firewall.cx/linux-knowledgebase-tutorials/system-and-network-services/829-linux-bind-introduction.html)
- Update and install bind
	- `sudo apt-get update`
	- `sudo apt-get upgrade`
	- `sudo apt-get install bind9 bind9utils bind9doc`
	- bind is a widely used domain name system software for your server to become a DNS for your network 
- Configure caching name server
	- `sudo nano /etc/bind/named.conf.options`
	- uncomment and change the lines 
	```
	//forwarders {
	//	0.0.0.0;
	//};
	```
	to 
	```
	forwarders {
		8.8.8.8;
		8.8.4.4;
	};
	```
- Change dns-nameservers so that bind will handle it 
	- `sudo nano /etc/network/interfaces`
	- change to 
	```
	auto enp0s3
	iface enp0s3 inet static
		address 172.20.240.23
		netmask 255.255.255.0
		gateway 172.20.240.254
		dns-nameservers 172.0.0.1
	```
- Refresh 
	- `sudo ip addr flush enp0s3`
	- `sudo systemctl restart networking.service`
	- `sudo systemctl restart bind9`
	- `ping www.google.com`
	
