# Broadcast Traffic Analysis Lab

## Overview
Demonstrated passive network reconnaissance by capturing and analyzing broadcast traffic on a local subnet with tcpdump. Unlike active scanning (ping sweeps, Nmap), this technique sends zero packets toward the target and is therefore undetectable by network monitoring tools.

## Scenario
Simulated an early-stage red team engagement where the goal was to map live hosts on the local segment without triggering IDS/IPS alerts. Passive sniffing of ARP broadcasts reveals active hosts as they communicate, without touching them directly.

## Lab Environment
- Sniffer: Kali Linux (10.0.2.15)
- Interface: eth0
- Target subnet: 10.0.2.0/24 (VirtualBox NAT)
- Tool: tcpdump

## Commands Used

### Capture ARP broadcasts
sudo timeout 20 tcpdump -i eth0 -n arp > arp_capture.txt 2>&1 &

### Generate traffic (to trigger broadcasts)
sudo arp-scan 10.0.2.0/24

## Other Useful Filters (from the lab text)
- tcpdump -i eth0 arp                     # ARP broadcasts
- tcpdump -i eth0 udp port 137            # NetBIOS Name Service (Windows)
- tcpdump -i eth0 udp port 5355           # LLMNR (Windows)
- tcpdump -i eth0 udp port 5353           # mDNS (Linux/macOS)
- tcpdump -i eth0 broadcast               # All broadcast traffic

## Results
- **512 ARP packets captured** in a 20-second window
- Sample lines:

## Analysis
The capture reveals two active hosts on the segment:
- **10.0.2.2** - VirtualBox NAT gateway (MAC 52:55:0a:00:02:02)
- **10.0.2.3** - VirtualBox DNS (MAC 52:55:0a:00:02:03)

On a real corporate LAN, this same technique would reveal:
- Windows domain controllers broadcasting NetBIOS names
- Windows workstations resolving via LLMNR
- Linux/macOS systems advertising via mDNS (Avahi)
- Printers, IoT devices, and other ARP-active hosts

## Why Passive Sniffing Matters

| Aspect | Active Scan (ping/nmap) | Passive Sniff (tcpdump) |
|--------|-------------------------|-------------------------|
| Packets sent to target | Yes | No |
| Detectable by target | Yes (obvious) | No (invisible) |
| Works when ICMP blocked | No | Yes |
| Reveals OS fingerprints | No (needs follow-up) | Yes (protocol behavior) |
| Detectable by network IDS | Often | Rarely |

## Limitations
1. Broadcast traffic is only visible **within the same subnet/VLAN** - not across routers.
2. Requires the attacker to already be on the network (post-initial-access).
3. Only reveals hosts that broadcast. Silent hosts (no NetBIOS, no mDNS, no ARP) may not appear.

## Key Takeaways
1. Passive sniffing is the stealthiest form of host discovery - no packets sent, no logs generated.
2. ARP is a goldmine for host discovery because every active host must use it.
3. Windows hosts leak hostnames via NBNS (UDP 137) and LLMNR (UDP 5355).
4. Linux/macOS hosts leak hostnames via mDNS (UDP 5353).
5. For defenders: this is why network segmentation and broadcast suppression matter.

## Evidence
- arp_capture.txt - Trimmed tcpdump output (first 100 packets)

## Tools Used
- tcpdump
- arp-scan (to generate traffic)
