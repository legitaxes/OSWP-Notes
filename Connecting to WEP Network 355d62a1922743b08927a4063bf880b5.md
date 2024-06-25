# Connecting to WEP Network

- Create config file
    - `sudo vi <filename>.conf`
        
        ```bash
        network={
        	 ssid="hotel_wifi"
        	 key_mgmt=NONE
        	 wep_key0=KEY_Withot_Colons
        	 wep_tx_keyidx=0
        }
        ```
        
- Connect to open network using config file
    - `sudo wpa_supplicant -Dnl80211 -iwlan1 -c <filename>.conf`
- Obtain AP IP Address
    - `sudo dhclient -v wlan1`