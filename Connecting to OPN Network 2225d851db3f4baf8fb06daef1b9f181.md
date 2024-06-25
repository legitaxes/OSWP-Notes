# Connecting to OPN Network

- Monitor the network to find out the AP name
    - `sudo airodump-ng wlan0mon`
- Once target found, lock in on it using the specific BSSID
    - `sudo airodump-ng --bssid <BSSID> -c <channel> wlan0mon`
- Create a config file
    - `vi <filename>.conf`
    
    ```bash
    network={
    	ssid="AP NAME HERE"
    	key_mgmt=NONE
    }
    ```
    
- Connect to the AP manually
    - `sudo wpa_supplicant -Dnl80211 -iwlan1 -c <filename>.conf`
- Using another terminal, run the DHCP and find the IP address
    - `sudo dhclient -v wlan1`
- Based on the above command, able to see AP’s login IP Address

In the case where you cannot access the page, it could be because you need to spoof your MAC address to one of the clients connected to the target AP

```bash
systemctl stop network-manager
sudo ip link set wlan1 down
sudo macchanger -m <MAC ADDR OF CLIENT> wlan1
sudo ip link set wlan1 up
```

- Re-run the connection to AP
    - `sudo wpa_supplicant -Dnl80211 -iwlan1 -c <filename>.conf`
    - `sudo dhclient -v wlan1`