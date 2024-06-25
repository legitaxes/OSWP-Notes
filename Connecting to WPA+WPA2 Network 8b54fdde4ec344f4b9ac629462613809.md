# Connecting to WPA+WPA2 Network

- Create config file
    - `sudo vi <filename>.conf`
        
        ```bash
        network={
        	 ssid="hotel_wifi"
        	 key_mgmt=WPA-PSK
        	 psk="hotel_password"
        	 priority=100
        }
        ```
        
- Connect to open network using config file
    - `sudo wpa_supplicant -iwlan1 -c <filename>.conf`
    
    *note: omitted -Dnl80211 (?)*
    
- Get AP IP Address
    - `sudo dhclient -v wlan1`