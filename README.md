# RF practical
## The aims of your practical or hypothesis
## Equipment, tools or code used

### On the host (macOS + VMWare Fusion):

VMWare -> VM Settings -> USB -> set the guest to not share bluetooth devices with Linux

```
# remove the CSR device from the host kernel so it can be used in VMWare
sudo kextunload -b com.apple.iokit.CSRBluetoothHostControllerUSBTransport

```

### On the guest (Kali 2020.1)

Start and add the CSR and Alfa devices
```
sudo airmon-ng check kill
sudo airmon-ng start wlan0
sudo bettercap
```

(bettercat) Enable the wifi dump
```
set wifi.interface wlan0mon
wifi.recon on
```

## Methodology
## Observations
## Analysis and conclusions 
