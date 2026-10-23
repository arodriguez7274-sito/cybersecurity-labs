# Lab 03: USB Wi-Fi Adapter Setup & Monitor Mode

**Date:** 2026-10-23
**Platform:** Kali Linux (VirtualBox)
**Difficulty:** Medium
**Target:** Ralink RT5370 USB Adapter
**Skill Focus:** Hardware Passthrough · Driver Installation · Monitor Mode

---

## Objective

The goal of this lab was to successfully pass a physical USB Wi-Fi adapter through a VirtualBox hypervisor to a Kali Linux virtual machine, verify the hardware is recognized, and enable Monitor Mode to capture raw 802.11 wireless traffic.

## Setup & Troubleshooting

Getting the USB adapter to work inside a VM required several troubleshooting steps:

1. **VirtualBox USB Settings:** Initially, the Kali VM could not see the physical adapter. I had to shut down the VM, go into VirtualBox settings, enable the USB Controller (xHCI), and create a USB Device Filter to lock the specific Ralink adapter to the VM.
2. **Driver Verification:** Unlike the instructor's Realtek adapter, my adapter uses a **Ralink RT5370** chipset. I used the `lsusb` command to verify the chipset and confirmed that the `rt2800usb` driver is built directly into the Kali kernel, meaning no external driver download was required.
3. **Interface Management:** The `ifconfig` command initially showed the interface as DOWN with no IP address. Running `sudo airmon-ng check kill` was necessary to stop NetworkManager and wpa_supplicant from interfering with the adapter.

## Execution & Results

### 1. Enabling Monitor Mode
**Command:**
`sudo airmon-ng start wlan0`

**Result:** Successfully created a new monitor mode interface named `wlan0mon`. Verified using `iwconfig`.

### 2. Capturing Traffic
**Command:**
`sudo airodump-ng wlan0mon`

**Result:** Successfully scanned the local airwaves. The output displayed nearby BSSIDs (routers), their signal strength (PWR), channels, and encryption types (WPA2/WPA3). 

### 3. Packet Analysis
Opened Wireshark and captured live traffic on the `wlan0mon` interface, observing 802.11 Beacon frames and Probe Requests.

## Lessons Learned

* **Hardware passthrough requires USB filters:** Simply plugging a USB into a computer running a VM isn't enough. The hypervisor must be explicitly told to hand that specific device over to the guest OS, otherwise, the host OS will keep it.
* **Not all drivers need downloading:** Linux has excellent built-in driver support for many legacy Wi-Fi chipsets (like the Ralink RT5370). Identifying the exact chipset with `lsusb` saves time compared to blindly downloading drivers.
* **Background services interfere:** Tools like NetworkManager are great for normal daily use, but they actively fight tools like `aircrack-ng` for control of the Wi-Fi card. Knowing to use `airmon-ng check kill` is a crucial step in wireless pentesting.

## References

- Aircrack-ng Documentation: https://www.aircrack-ng.org/
- Wireshark: https://www.wireshark.org/
