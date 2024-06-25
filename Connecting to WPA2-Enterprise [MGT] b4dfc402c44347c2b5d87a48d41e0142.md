# Connecting to WPA2-Enterprise [MGT]

- Create config file
    - `sudo vi <filename>.conf`
        
        ```bash
        network={
          ssid="NetworkName"
        	scan_ssid=1
          key_mgmt=WPA-EAP
          identity="Domain\username"
          password="password"
          eap=PEAP
          phase1="peaplabel=0"
        	phase2="auth=MSCHAPV2"
        }
        ```
        
- Connect to network using config file
    - `sudo wpa_supplicant -iwlan1 -c <filename>`
    
    *note: omitted -Dnl80211 (?)*
    
- Get AP IP adress
    - `sudo dhclient -v wlan1`