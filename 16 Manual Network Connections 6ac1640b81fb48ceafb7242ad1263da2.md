# 16. Manual Network Connections

Need to separate network interfaces from the ones we use for pentesting

Most common wireless client in linux is wpa_supplicant, commonly used by network managers across linux distributions to connect to wifi networks. can be used via cmd or wpa_cli or with config files containing settings of network. 

In sample configuration file, each network connection is defined within network item

![Untitled](Untitled%2014.png)

Allows system to connect to open network called hotel_wifi and wpa_supplicant is instructed in the third line to scan for ssids first 

Connecting to WPA-PSK is bit more involved as need to add 2 parameters inside network item

![Untitled](Untitled%2015.png)

wpa_supplicant auto choose between TKIP and CCMP based on availability, possible to force one or the other by adding pairwise=CCMP or pairwise=TKIP to config

wpa_supplicant supports WPA3, OWE and can handle WPA enterprise networks as well. 

Using the example in the above, can create a file called wifi-client.conf. Now have to start wpa_supplicant with couple parameters. To connect to network, have to start wpa_supplicant with network interface using -i and config file with -c on the created file from above.

`sudo wpa_supplicant -i wlan0 -c wifi-client.conf`

![Untitled](Untitled%2016.png)

After confirming can successfully connect to network. can append -B to the command above to run in the background

Once connected, can usually request for DHCP lease using dhclient

`sudo dhclient wlan0`

Setting up an access point requires two distinct network interfaces, and involves five steps:

1. Configure Internet access on the system.
2. Set up a static IP for the wireless interface.
3. DHCP server set up, to provide automatic IP configuration for Wi-Fi clients.
4. Add routing to provide Internet access to the Wi-Fi clients.
5. Configure the Wi-Fi interface in AP mode.

Setting up static IP on AP Wireless interfaces

`sudo ip link set wlan0 up`

`sudo ip addr add 10.0.0.1/24 dev wlan0`

Setting up DHCP server using dnsmasq

- create the following dnsmasq.conf file
    
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
    listen-address=10.0.0.1
    
    # DHCP range
    dhcp-range=10.0.0.100,10.0.0.199,12h
    dhcp-lease-max=100
    # Router: wlan0
    dhcp-option=option:router,10.0.0.1
    dhcp-authoritative
    
    # DNS: Primary and secondary Google DNS
    server=8.8.8.8
    server=8.8.4.4
    ```
    

listen-address and router option is address of wlan0 interface. DHCP-range will be a range of IP addresses to provide to clients following by length of lease. Two server entries are upstream DNS servers

run dnsmasq with `--conf-file` 

`sudo dnsmasq --conf-file=dnsmasq.conf`

Can confirm it started correctly 

`sudo tail /var/log/syslog | grep dnsmasq` 

Enable routing and add a few firewall rules and allow clients to reach internet

`echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward`

Setup hostapd.conf file with the following config

```bash
interface=wlan0
ssid=BTTF
channel=11

# 802.11n
hw_mode=g
ieee80211n=1

# WPA2 PSK with CCMP
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_passphrase=GreatScott
```

Run hostapd with single parameter with its config file

`sudo hostapd hostapd.conf`