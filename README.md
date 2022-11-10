# Pi-hole

This repository provides the basic docker compose artifacts to:

1.  Pull the latest pi-hole + unbound image from dockerhub and stand up and environment.  This uses the cbcrowe/pihole one container image which is based on the official pi-hole image with extra build steps to add in unbound as a recursive DNS server.  More info at https://github.com/chriscrowe/docker-pihole-unbound.
2.  The docker-compose.yml file contains references to environmental variables. You will need to create your own .env file and populate it with the paramters for your own set up.
3.  From terminal use the command "sudo docker compose up -d" to pull and start up the pi-hole environment.
4.  You will need to configure the pi-hole host to use a static IP address.  
5.  As well the host DNS settings will need to be configured to use 127.0.0.1 as the primary DNS.  If you have another pi-hole, you can use this as the secondary DNS server.
6.  To configure individual clients/devices to use pi-hole, add it's IP into their DNS settings.
7.  To configure pi-hole network wide, you will need to update your router's DHCP settings to use the pi-holes.  Note for Asus routers, this is done through the DHCP settings (which is what the router hands out to client devices) and NOT through the WAN settings (which is what it uses itself).  

### To Deploy a 2nd Pi-hole
To deploy a another pi-hole to hand out as a secondary DNS, you will need to update the .env file with the particulars of the 2nd pi-hole.  

## Additional Steps

### For Ubuntu
(https://github.com/pi-hole/docker-pi-hole)

Installing on Ubuntu or Fedora
Modern releases of Ubuntu (17.10+) and Fedora (33+) include systemd-resolved which is configured by default to implement a caching DNS stub resolver. This will prevent pi-hole from listening on port 53. The stub resolver should be disabled with: 

    sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf

This will not change the nameserver settings, which point to the stub resolver thus preventing DNS resolution. Change the /etc/resolv.conf symlink to point to /run/systemd/resolve/resolv.conf, which is automatically updated to follow the system's netplan:

    sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf' 
    
After making these changes, you should restart systemd-resolved using 
    
    systemctl restart systemd-resolved

### For Raspberry PI
(https://www.raspberrypi.com/documentation/computers/configuration.html)

You may get a unable to resolve DNS error when trying to trigger a Gravity update.  To resolve this, the host of the pi-hole image needs to be configured to use "127.0.0.1" as it's primary dns.  If you have 2 pi-hole's, you can configure the other pi-hole as the secondary DNS.  This also ensures that if one pi-hole DNS goes down, it won't leave the other one uanble to resolve DNS's.

Note the raspberry pi OS uses resolvconf.conf to generate resolve.conf - means you CANNOT directly edit resolv.conf.  Instead you should configure the DNS (and IP) setting with the included utility raspi-config   Note you can add in 2 DNS server IP's with a " " space in between.

### For Asus Routers
(https://www.reddit.com/r/pihole/comments/mduhv3/chicken_and_egg_problem_with_asus_router_wan/)

As of writing, the latest Asus firmware (3.0.0.4.386.49703) only allows you to specify 1 DNS server in the DHCP settings.  By default it will give out it's own IP as the secondary one (and will forward DNS requests off to the ISP provided DNS servers).  To prevent this you can either run the Merlinwrt firmware (as it has an option to disable this) or SSH into the router and execute the following commands (where yyy.yyy.yyy.yyy is your secondary pi-hole - BUT do at your own risk though):

    nvram set dhcp_dns2_x=yyy.yyy.yyy.yyy | nvram commit
    service restart_dnsmasq

### Domain 
If you are using a domain name (ie home.arpa), make sure that both pi-hole hosts have been properly configured with the domain name.  

In the case of the raspbian, it was able to get the domain name from the router as long as the network connection was set to static IP and DNS's, but to automatically fill in (from the router) other values (ie the domain).  

In the case of ubuntu, the network connection gui does not have an option to automatically fill in the missing values from the router when using a manual/static connection.  One work around for this is to add in home.arpa into the Domain Search.  This can be done through the Show Apps=>Utilities=>Advanced Network Configuration.

To test if it's properly pointed to the domain, you can run a nslookup from both pi-hole hosts (ie nslookup mywordpress).  They should both be able to resolve the short & full names and always return the full name (mywordpress.home.arpas).

