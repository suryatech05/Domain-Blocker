![image](https://github.com/suryatech05/Domain-Blocker/assets/120020431/05beec62-ea1e-4430-956c-8ade2bc3f144)


**Introduction**

  Pi-hole is a network-level ad blocker that functions as a Domain Name System (DNS) sinkhole. It is designed to be installed on a Raspberry Pi or other Linux-based server to provide network-wide ad blocking, thereby reducing the amount of internet traffic and increase browsing speed.
When Pi-hole is installed on a network, it acts as a filter between a user's device and the internet, blocking requests to known ad-serving domains and preventing them from loading. This means that domains are not only blocked in the user's browser, but also on any device that connects to the network.
The rise of online advertising has led to an increase in ads, tracking, and privacy violations. This has resulted in a demand for efficient ad-blocking solutions that can improve online security and enhance user experience. Pi-hole is a network-level ad-blocker that functions by blocking DNS requests for blacklisted domains.
The goal of this project is to develop a Pi-hole-based domain blocker that provides effective ad-blocking, enhances online privacy for users, increases the network speed. The domain blocker should be easy to install, manage and should provide users with granular control over the domains they want to block for parenting network. Additionally, the domain blocker should be customizable and allow users to add their own custom blacklists.


**Implementation**

⦁	Install docker packages:

      apt install docker.io

⦁	Imaging the docker container from pinhole administrators

      blockie.sh (Code for imaging container)

⦁	Download required DNS settings from the image.

      #!/bin/bash
  
      # https://github.com/pi-hole/docker-pi-hole/blob/master/README.md
  
      docker run -d \
          --name pihole \
          -p 53:53/tcp -p 53:53/udp \
          -p 80:80 \
          -p 443:443 \
          -p 8080:8080 \
          -e TZ="America/Chicago" \
          -v "$(pwd)/etc-pihole/:/etc/pihole/" \
          -v "$(pwd)/etc-dnsmasq.d/:/etc/dnsmasq.d/" \
          --dns=127.0.0.1 --dns=1.1.1.1 \
          --restart=unless-stopped \
          thenetworkchuck/networkchuck_pihole
      
      printf 'Starting up pihole container '
      for i in $(seq 1 20); do
          if [ "$(docker inspect -f "{{.State.Health.Status}}" pihole)" == "healthy" ] ; then
              printf ' OK'
              echo -e "\n$(docker logs pihole 2> /dev/null | grep 'password:') for your pi-hole: https://${IP}/admin/"
              exit 0
          else
              sleep 3
              printf '.'
          fi
      
          if [ $i -eq 20 ] ; then
              echo -e "\nTimed out waiting for Pi-hole start, consult check your container logs for more info (\`docker logs pihole\`)"
              exit 1
          fi
      done;
      © 2020 GitHub, Inc.

⦁ After the code execution Pi-hole would give us authentication credentials and the IP address of the Pi-hole would be same as our system network IP address.
⦁	For managing domains we need to configure our main DNS server as our blocker admin IP.
⦁	To do it we write a conf file in /etc folder for DNS utilities in Linux
                        
      cd /etc
                        
      gedit resolv.conf
⦁	Make nameserver same as blocker IP address so it uses blocker as DNS server. search local domain
nameserver 192.168.__.___ (Same as blocker pihole IP)


**Setting pi hole as recursive DNS server solution**
⦁	Install unbound service using the command:
              
      apt install unbound
              
⦁	Configure unbound service○Listen only for queries from the local Pi-hole installation (onport 5335)
⦁	Listen for both UDP and TCP requests

⦁		Verify DNSSEC signatures, discarding BOGUS domains○Apply a few security and privacy tricks

⦁	The configuration file is present empty in the directory named:  "/etc/unbound/unbound.conf.d/"

⦁		In the conf file set the parameters for features of verbosity, interfaces, dnssec, edns buffer size, the local IP ranges with an unused port number (In my case I have       used 5335 with interface 127.0.0.1 represented as 127.0.0.1#5335.

⦁		Now in pihole/admin page goto Settings->DNS option add the Custom Upstream DNS servers tab “Custom1 (IPv4)” as 127.0.0.1#5335 and check the box. Finally, Save the           changes at the bottom corner of the page

⦁	  Open pi-hole.conf in the directory using the commands:

    cd /etc/unbound/unbound.conf.d/

    gedit pi-hole.conf

⦁   Write the below code in the file (pi-hole.conf):


    server:
        verbosity: 0
        interface: 127.0.0.1
        port: 5335
        do-ip4: yes
        do-udp: yes
        do-tcp: yes
        do-ip6: no
        prefer-ip6: no
        harden-glue: yes
        harden-dnssec-stripped: yes
        use-caps-for-id: no
        edns-buffer-size: 1232
        prefetch: yes
        num-threads: 1
        so-rcvbuf: 1m
        private-address: 192.168.0.0/16
        private-address: 169.254.0.0/16
        private-address: 172.16.0.0/12
        private-address: 10.0.0.0/8
        private-address: fd00::/8
        private-address: fe80::/10
    

**Update the gravity file.**
⦁		After the adding of blacklists update the gravity file of pihole using command: 
        
    pihole -g
        
⦁		The other way is to manually update the gravity file by undergoing Tools-> Update Gravity in the web interface

⦁   Now open any browser of your interest and type down the Pi-hole IP address and in the "Domains" tab in the left panel of Pi-hole and add the blacklist domains that you want to block.

⦁   We can block the domains that are being accessed frequently by manual "Blacklist" button and "Whitelist" button for unblocking.


**Conclusion**
  
  I have configured and customized the utilities of Pi-hole and blocked some domains from the network. There are several other advanced tweaks such as Pihole Host file, dnsmasq in the router, DNS over HTTPS, Moving Query logs to RAM to save complex analysis, Configuring whole Home router ad blocking which would be continued in several cycles.
