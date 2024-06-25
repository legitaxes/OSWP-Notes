# Using Aircrack-ng to Crack Hashes

To start, have to capture a WPA 4-way handshake between an AP and a real client. Will be used throughout the course. 

4-way handshake

- contains necessary information to crack passphrase

steps to capture network

1. put wlan0 interface to monitor mode
2. identify the channel of target AP and gather its BSSID to limit capture with airodump-ng
    
    `sudo airodump-ng wlan0mon`
    
    BSSID, Channel operating on, and AUTH method used captured
    

![Untitled](Untitled%2026.png)

       *note: aircrack-ng does not work on MGT auth method*

*note 2: airodump-ng only scans 2.4GHz by default. to scan 5GHz channels use `sudo airodump-ng wlan0mon --bang abg`* 

1. Next update airodump-ng command to reflect the `BSSID` and `channel` as well as its `SSID` of the AP to capture the packets of the AP connection. Save the captured data into a file using `-w` 
    1. ensure your wlan0mon is on the same channel as your targeted AP
    
    *impt to be on the same channel as the AP if not deauth and handshake capture will **fail.*** 
    
    `sudo airodump-ng -c 3 -w wpa --essid wifu --bssid 34:08:04:09:3D:38 wlan0mon` 
    
    ***Highly recommend to keep this command running while running the next command in 4.***
    
2. **Concurrently run aireplay-ng** to deauth or disconnect target AP and its connected client on another terminal
    
    `sudo aireplay-ng -0 1 -a 34:08:04:09:3D:38 -c 00:18:4D:1D:A8:1F wlan0mon`
    
    - -0 is deauth attack
    - -a indicates target AP MAC-addr
    - -c indicates connected client MAC-addr

1. Aireplay-ng verifies BSSID then deauth the clients. Once the client tries to reconnect, airodump-ng will capture the 4-way handshake packets. Let it run for a bit to capture the traffic between client and AP. 
    1. notification will appear if handshake is captured. if not captured, signal is too weak or too strong. Only portion of handshake was captured. just wait out
    2. Some wireless drivers ignore directed deauth and only responds to broadcast deauth.
        1. to fix this, rerun the command without -c 
    3. if using 802.11w, unencrypted deauth frames ignored. Wait for client to connect
    4. device simply did not reconnect or was already out of range of AP

1. Once handshake captured, then run aircrack-ng on the captured file
    
    `sudo aircrack-ng -w ROCKYOU.txt -e wifu -b 34:08:04:09:3D:38 wpa-01.cap` 
    
    `-w` specify file of password list
    
    `-e` indicate ESSID
    
    `-b` specify BSSID (mac addr)
    
    ![shows the password under KEY FOUND!](Untitled%2027.png)
    
    shows the password under KEY FOUND!
    
    1. Once aircrack-ng cracked, confirm the key is correct using **airdecap-ng.** Run it against the file we captured traffic.
        
        `airdecap-ng -b 34:08:04:09:3D:38 -e wifu -p 12345678 wpa-01.cap`
        
        - Have to specify `-e` in decrypting WPA.
        - Use `-p` to specify cracked passphrase
        
        *it is possible that result may indicate a “0” packets decrypted even if passphrase is correct.*
        
    
    1. Smarter way to bruteforce AP is to look at the manufacturer of AP based on the first 3 bytes of the mac address. Then search up the default password of the AP or patterns to create useful wordlist
    
    1. There’s also many tools that can expand our wordlists.
        1. John
        2. Crunch
        3. RSMangler
    
     Check next section for continuation