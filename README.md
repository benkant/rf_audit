# RF practical
## Research aim

The purpose of this research is to evaluate the effectiveness of the `Internet kill switch` (herein IKS) functionality of the _Mudi (GL-E750)_ by _GL.iNet_ [1](#notes).

The IKS is defined thus [2](#notes)

> [...] if the VPN client is not running, the clients are Not Allowed to access the Internet

## Equipment, tools, environment setup

### Device under audit
| Name | `charlie` |
| --- | --- |
| Make | GL Technologies (HK) Limited & Microuter Technologies Limited trading as _GL-iNet_ [3](#notes) |
| Manufacturer | Shenzhen Guanglianzhitong Tech Co., Ltd. [4](#notes) |
| Model | Mudi (`GL-E750C6-E`) [4](#notes) |
| Firmware | As shipped (version: 3.100, compile time: `2019-12-16 15:45:15`) [5](#notes) |
| VPN Client | OpenVPN, UDP, `ch-uk-01.protonvpn.com` [6](#notes) |
| IKS | Enabled |
| Network Mode | Router [7](#notes) |
| WiFi Settings | 2.4GHz WPA2-PSK [8](#notes) |
| 802.11 BSSID | `94:83:C4:02:1A:35` |
| 802.11 EDDID | `charlie` |

### Device being monitored
| Name | `DuckSeason` |
| --- | --- |
| Make | Huawei Technologies Co., Ltd. |
| Model | HUAWEI Home Gateway (HG659) |
| 802.11 BSSID | 5C:03:39:CC:B1:78 |
| 802.11 ESSID | RabbitSeason |

### Traffic source to be protected by `charlie`
| Name | `strike` |
| --- | --- |
| Make | Apple Inc. |
| Manufacturer | Hon Hai Precision Industry Co. Ltd. trading as Foxconn [9](#notes) |
| Model | iPhone 11 Pro |
| VPN | Disabled |
| OS | iOS 13.3.1 |
| 802.11 MAC | f8:4e:73:84:a7:b8 |
| IP address when connected directly to `DuckSeason` | 192.168.1.3 |
| IP address when connected to `DuckSeason` via `charlie` | 192.168.8.145 |

### Auditing tools
| Name | `hollywood` |
| -- | -- |
| OS | Kali 2020.1 x64 |
| WiFi Card | Qualcomm Atheros AR9271 |
| Software | `aircrack-ng` suite, `wireshark` |

#### Common Setup

For both _control (null)_ and _research_ environments:

_hollywood_
```
sudo airmon-ng check kill
sudo airmon-ng start wlan0
```

**Environments**

_Control_

`strike` -> `DuckSeason` -> WLAN

_Research_

`strike` -> `charlie` -> `DuckSeason` -> WLAN

## Methodology

We capture packets in different environment configurations and compare them in order to determine whether the Mudi (`charlie`) protects network traffic.

### Definitions

We define `charlie`'s IKS:

1. *ineffective* if `aircrack-ng` is able to retrieve `strike`'s TCP/IP packets when IKS is enabled.
2. *effective* if `aircrack-ng` is unable retrieve `strike`'s TCP/IP packets with IKS is enabled[11](#notes)

We use a `wireshark` filter to determine whether packets were captured by `aircrack-ng`:
| Case | `wireshark` filter |
| -- | -- |
| A. `strike` connected directly to `DuckSeason` | `wlan.addr==f8:4e:73:84:a7:b8 && ip` |
| B. `strike` connected to `DuckSeason` via `charlie` | `wlan.addr==f8:4e:73:84:a7:b8 && ip` |

**Null hypothesis**: We use a control[10](#notes) to determine `aircrack-ng` is able to retrieve `strike`'s TCP/IP packets when not protected by `charlie`. Packets being present in cases A and B form the null hypothesis.

**Alternative hypothesis**: No display of TCP/IP packets[11](#notes) from `strike` when protected by `charlie`. This is the rejection of the null hypothesis.

| Outcome | Condition |
| -- | -- |
| Null hypothesis is rejected | Case (A) shows packets and case (B) shows no packets and therefore IKS is deemed effective. |
| Null hypothesis is accepted | Case (A) shows packets and case (B) shows packets and therefore IKS is deeemed ineffective. |
| Inconclusive | Case (A) shows no packets. |

### Method to capture and analysis of control data for null hypothesis

1. Connect `strike` to `RabbitSeason`
2. Capture packet TX/RX for `RabbitSeason`
```
sudo airodump-ng -w cap/RabbitSeason.pcap --output-format pcap -a --bssid 5C:03:39:CC:B1:78 --channel 11 wlan0mon
```
3. Open `RabbitSeason.pcap-01.cap` in `wireshark` with filter `wlan.addr==f8:4e:73:84:a7:b8 && ip`
4. Examine packets[10](#notes)
5. Existence of packets supports case (A).

### Method to capture and analysis of control data for alternative hypothesis

1. Connect `strike` to `charlie`
2. Capture packet TX/RX for `RabbitSeason`
```
sudo airodump-ng -w cap/RabbitSeason-with-charlie.pcap --output-format pcap -a --bssid 5C:03:39:CC:B1:78 --channel 11 wlan0mon
```
3. Open `RabbitSeason-with-charlie.pcap-01.cap` in `wireshark` with filter `wlan.addr==f8:4e:73:84:a7:b8 && ip`
4. Examine packets[11](#notes)
5. A lack of packets supports case (B).

## Observations

Significant effort was required to setup the environment such that IP packets could be monitored. This was likely due to VMWare's USB driver being unable to keep up with high speed 802.11 protocols. In order to mitigate this the device being monitored, `RabbitSeason`, needed to be dropped to 802.11b.

There is a significant amount of work that can, and should, be done to further audit the Mudi device in order to validate the vendor's claims and the devices many features:
- the physical rocker switch (to enable, say, Tor)
- OpenVPN Server
- WireGuard Client and Server
- other modes of WLAN access for the Mudi (tethering, 4G)

## Analysis and conclusions

Cases (A) and (B) were observed when following the above [methodology](#methodology). We therefore can reject the null hypothesis and accept the alternative hypothesis: the Mudi device under audit's IKS when used with OpenVPN protected IP traffic from interception when monitoring the ultimate 802.11 access point, `RabbitSeason`.

## Notes
1. "GL-E750 / Mudi - GL.iNet" GL.iNet, accessed February 24, 2020,
	https://www.gl-inet.com/products/gl-e750/
2. "Internet Kill Switch - GL.iNet Docs" GL.iNet Docs, February 24, 2020,
	https://docs.gl-inet.com/en/3/app/internet_kill_switch/
3. "Contacts - GL.iNet", accessed February 24, 2020
	https://www.gl-inet.com/contacts/
4. GL-E750 Packaging, photograph by author, February 24, 2020,
	https://raw.githubusercontent.com/benkant/rf_prac/master/img/package_detail.png
5. Device UI - Firmware, screenshot by author, February 24, 2020,
	https://raw.githubusercontent.com/benkant/rf_prac/master/img/fimware.png
6. Device UI - VPN Client, screenshot by author, February 24, 2020,
	https://raw.githubusercontent.com/benkant/rf_prac/master/img/vpn_config.png
7. Device UI - Network Mode, screenshot by author, February 24, 2020,
	https://raw.githubusercontent.com/benkant/rf_prac/master/img/network_mode.png
8. Device UI - WiFi Settings, screenshot by author, February 24, 2020,
	https://raw.githubusercontent.com/benkant/rf_prac/master/img/wifi_settings.png
9. Wikipedia - Foxxcon, accessed February 24, 2020,
	https://en.wikipedia.org/wiki/Foxconn
10. Wireshark - transmitted in the clear, screenshot by author, February 26, 2020,
	https://raw.githubusercontent.com/benkant/rf_prac/master/img/XXX.png
11. Wireshark - transmitted via Mudi & OpenVPN, screenshot by author, February 26, 2020,
	https://raw.githubusercontent.com/benkant/rf_prac/master/img/XXX.png

