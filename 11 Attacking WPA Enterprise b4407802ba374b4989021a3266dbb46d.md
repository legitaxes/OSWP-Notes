# 11. Attacking WPA Enterprise

WPA-based network access is a nightmare to manage in a larger organization

WPA3 is not always viable and set up can be complex for network administrators. 

**Solution**

WPA-Enterprise is much better solution as it enables user auth against central database, making protecting access to wireless network much more manageable. 

- also removes the danger of attackers sniffing on network as they could be in PSK.
- If one device gets compromised or if one employee leaves, only one account is needed to be revoked instead of revisiting every device on the network.
- Various auth schemes can be used as well and security can be very strong
- Although WPA enterprise networks seems impenetrable, **small mistakes can compromise the whole network**

Even though it uses 4 way handshakes, attacking WPA enterprise is completely **different from attacking preshared keys and will need a different set of tools** 

EAP-TLS 

- one of the most secure auth method, uses certificates on server side and client side instead of login and passwords.
- server and client mutually auth each other

EAP-TTLS 

- uses TLS, does not need client certificates
- creates tunnel then exchanges credentials using one of the few possible different inner methods (phase 2) such as Challenge-Handshake Authentication Protocol (CHAP), Authentication Protocol (PAP), Microsoft CHAP (MS-CHAP), or MS-CHAPv2

PEAP

- creates TLS tunnel before credentials exchanged
- different methods can be used within PEAP, MS-CHAPv2 is a commonly used inner method
- **differs on how data exchanged in TLS tunnel**

---

[PEAP Exchange](PEAP%20Exchange%20f2136d9900e14c76a2871a9593395104.md)

[Attack [Impt]](Attack%20%5BImpt%5D%2060a45434bc37476f856db5950ea8dc19.md)

---