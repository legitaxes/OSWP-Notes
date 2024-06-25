# Airodump-ng

Used to capture raw 802.11 frames. Useful for collecting WEP IVs or WPA/WPA2 handshakes

Can export files in various formats. Can create custom scripts, and integrate with other tools easily

**Most Often Used Options**

| OPTION | DESCRIPTION |
| --- | --- |
| -w <prefix> | Saves the capture dump to the specified filename |
| --bssid <BSSID> | Filters Airodump-ng to only capture the specified BSSID |
| -c <channel(s)> | Forces Airodump-ng to only capture the specified channel(s) |

**Sniffing w/ Airodump-ng**

`sudo airodump-ng wlan0mon -c <channel-id>`

![Untitled](Untitled%2017.png)

*Output is separated into 2 sections.*

1. Top portion shows detected APs along with encryption used and SSID
2. Lower portion shows stations sending frames and associated AP

First line indicates **current channel**, **elapsed sniffing time** and **current date and time**, and if there’s an indication of **WPA handshake sniffed**, it will displayed the AP and the BSSID [MAC-Address]

![Untitled](Untitled%2018.png)

Lower portion presents a list of devices. Station sending frames listed in **STATION** column. If stations are connected to an AP, the **SSID** will be displayed else shows **(not associated)**

Cannot send data and listen to the network at the same time. Every time a data being sent, cant listen for frames being transmitted for that interval

Part of the Initial Recon, want to determine SSID of target AP and its channel so that we can focus specifically for that

To sniff data of specific AP on given cahnnel, add `--bssid` to `airodump-ng` command. Use `-w` to output to a file

Like this:

`sudo airodump-ng -c 3 --bssid 34:08:04:09:3D:38 -w cap1 wlan0mon`

*monitoring on channel 3’s BSSID as above and output to a file called cap1 on the wlan0mon interface. filtering to show only one SSID*

![Untitled](Untitled%2019.png)

GUI Tools for Aircrack-ng:

- Fern Wifi Cracker
- WiFite
- GISKismet
- Airgraph-ng

**Output File Format**

Use `--output-format` to specify output file format

```
possible format type: pcap, ivs, csv, gps, kismet, netxml, logcsv
```

**Navigate Airodump-ng**

`space` freezes screen

**`tab` enables scrolling and can navigate with `arrow up/down`** 

`m` cycles through color options

`a` cycles thru different display options

`s` cycles through different sorting options

**Troubleshooting**

*Little or no Data captured*

- Make sure to sniff on specific channel using `-c`
- Make sure wireless card is in `monitor` mode
- Ensure no network manager causing interference. Use `sudo airmon-ng check` to check

*Stops Capturing after short period of time*

- Connection manager is running on system which takes wireless card out of `monitor` mode. Run `sudo airmon-ng check kill` before putting wifi card on `monitor` mode