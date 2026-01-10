# Pager Passpoint Scanner

A Passpoint/Hotspot 2.0 reconnaissance tool for the WiFi Pineapple Pager. Detects and analyzes Passpoint-enabled access points, including support for 6GHz MBSSID networks.

## Features

- **True Passpoint Detection**: Identifies Hotspot 2.0 networks via IE 221 with HS2.0 OUI (50:6f:9a:10)
- **MBSSID Support**: Detects non-transmitted BSSIDs in 6GHz MBSSID beacons (IE 71)
- **ANQP Queries**: Actively queries Access Network Query Protocol data:
  - Domain Names (ID 268)
  - NAI Realms (ID 263)
  - 3GPP/PLMN Networks (ID 264)
  - Roaming Consortium OIs (ID 261)
  - Venue Name & URL (ID 258, 277)
  - HS2.0 Operator Friendly Name
- **RCOI Decoding**: Identifies known Roaming Consortium OIs:
  - OpenRoaming (Settlement-Free & Settled)
  - eduroam
  - Major carriers (AT&T, T-Mobile, Verizon)
  - Boingo, iPass, Cisco OpenRoaming
- **OpenRoaming Policy Decode**: Extracts policy details from OpenRoaming RCOIs:
  - Industry type (Enterprise, Education, Hospitality, etc.)
  - Level of Assurance (Baseline/Enhanced)
  - QoS Tier (Bronze/Silver)
  - Identity (Anonymous/Identified)

## Requirements

- WiFi Pineapple Pager
- Monitor mode capable WiFi adapter (wlan1mon)
- wpa_supplicant with interworking support

## Installation

### On WiFi Pineapple Pager

Copy the payload to the Pager's payload directory:

```bash
scp payload.sh root@172.16.52.1:/root/payloads/library/user/reconnaissance/passpoint_scanner/payload.sh
```

Or clone directly on the Pager:
```bash
cd /root/payloads/library/user/reconnaissance/
git clone https://github.com/WiFivomFranMan/Pager-Passpoint-Scanner.git passpoint_scanner
```

## Usage

### Via Pager UI
Navigate to: **Payloads > User > Reconnaissance > Passpoint Scanner**

### Headless Mode (SSH)
```bash
./payload.sh --headless
```

### Options
- `--headless` or `-H`: Run without Pager UI interaction
- `--channel N` or `-c N`: Scan specific channel only

## Output

Results are saved to `/root/loot/passpoint/passpoint_results_YYYYMMDD_HHMMSS.txt`

### Example Output
```
=== DETAILED RESULTS ===
----------------------------
AP: R1-Passpoint
BSSID: aa:bb:cc:dd:ee:ff
Channel: 132 | RSSI: -45dBm

  RCOI: 5A03BA0800: OpenRoaming (Settlement-Free) [Education,Anonymous,Bronze-QoS,Baseline-LoA]
  Venue: Educational (type 1) [eng] University Campus
  Operator: [eng] Campus WiFi
  NAI Realms: eduroam.edu, openroaming.org
  PLMNs: 310-410, 311-480
  Domains: eduroam.edu, openroaming.org
```

## How It Works

### Phase 1A: Beacon Scanning
- Switches to managed mode for `iw scan`
- Scans 2.4GHz, 5GHz (including DFS), and 6GHz bands
- Parses IE 221 for HS2.0 Indication (OUI 50:6f:9a type 0x10)
- Parses IE 71 (MBSSID) for non-transmitted Passpoint BSSIDs

### Phase 1B: SSID Grouping
- Groups discovered APs by SSID
- Prefers transmitted BSSIDs over MBSSID (ANQP queries only work on transmitted)
- Selects strongest signal for each SSID

### Phase 1C: ANQP Query
- Uses wpa_supplicant with `interworking=1` and `hs20=1`
- Sends GAS queries for ANQP elements
- Decodes hex responses into human-readable format

## Technical Details

### Passpoint Detection
- **IE 107 (Interworking)**: NOT a Passpoint indicator - just 802.11u
- **IE 221 with OUI 50:6f:9a type 0x10**: TRUE Passpoint/HS2.0 Indication

### MBSSID Limitation
Non-transmitted BSSIDs (from 6GHz MBSSID IE 71) cannot be queried for ANQP data via wpa_supplicant due to kernel limitations. The scanner detects these and marks them appropriately.

### RCOI Structure (OpenRoaming)
```
[BASE-24][N1][N2][N3][N4]

BASE-24:  5A03BA (Settlement-Free) or BAA2D0 (Settled)
N1:       bit3=LoA, bit1=QoS, bit0=PID
N2:       Industry type (0-B)
N3:       Credential lifetime (0=Long, 8=Short)
N4:       Reserved
```

## Known RCOIs

| Prefix | Organization |
|--------|--------------|
| 5A03BA | OpenRoaming Settlement-Free |
| BAA2D0 | OpenRoaming Settled |
| 001BC50460 | eduroam |
| 506F9A | Wi-Fi Alliance |
| F4F5E8 | Cisco OpenRoaming |
| 004096 | Boingo Wireless |
| 001BC5 | iPass/Pareteum |
| 001907 | AT&T |
| 00019E | T-Mobile |
| 0022F1 | Verizon |

## License

MIT License - See [LICENSE](LICENSE)

## Author

WiFivomFranMan

## Acknowledgments

- WiFi Pineapple community
- Wireless Broadband Alliance (OpenRoaming specification)
- eduroam community
