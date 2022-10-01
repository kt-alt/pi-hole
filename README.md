# Welcome to Pi-hole Git

This repository provides the basic docker compose artifacts to:

1.  Pull the latest pi-hole image from dockerhub and stand up and environment
2.  The docker-compose.yml file contains references to environmental variables. You will need to create your own .env file and populate it with the paramters for your own set up.
3.  From terminal use the command "sudo docker compose up -d" to pull and start up the pi-hole environment.
4.  You will need to configure the pi-hole host to use a static IP address.  
5.  As well the host DNS settings will need to be configured to use 127.0.0.1 as the primary DNS.  If you have another pi-hole, you can use this as the secondary DNS server.  

## Additional Steps

For Ubuntu (https://github.com/pi-hole/docker-pi-hole)

    Installing on Ubuntu or Fedora
    Modern releases of Ubuntu (17.10+) and Fedora (33+) include systemd-resolved which is configured by default to implement a caching DNS stub resolver. This will prevent pi-hole from listening on port 53. The stub resolver should be disabled with: 
    
        sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf
    
    This will not change the nameserver settings, which point to the stub resolver thus preventing DNS resolution. Change the /etc/resolv.conf symlink to point to /run/systemd/resolve/resolv.conf, which is automatically updated to follow the system's netplan:
    
        sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf' 
        
    After making these changes, you should restart systemd-resolved using systemctl restart systemd-resolved

For Raspberry PI (https://www.reddit.com/r/pihole/comments/mduhv3/chicken_and_egg_problem_with_asus_router_wan/)

    You may get a unable to resolve DNS error when trying to trigger a Gravity update.  To resolve this, the host of the pi-hole image needs to be configured to use "127.0.0.1" as it's primary dns.  If you have 2 pi-hole's, you can configure the other pi-hole as the secondary DNS.  Note the raspberry pi OS uses resolvconf.conf to generate resolve.conf - mean you CANNOT directly edit resolv.conf.  Instead you should configure the DNS (and IP) setting with the included utility raspi-config (https://www.raspberrypi.com/documentation/computers/configuration.html).