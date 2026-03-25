# Packet Capture on Cisco ASA

> **Applies to:** Cisco ASA firewalls (all versions with CLI access)

> **Key behavior:** Incoming packets hit the capture **before** any ACL, NAT, or other processing. Outgoing packets hit the capture **last**, right before being placed on the wire.

## Overview

The Cisco ASA supports inline packet capture from the CLI. You can filter by interface, protocol, source/destination, or reference an ACL. Captures can be downloaded as PCAP files via ASDM or CLI.

## Quick Start (Inline Match)

Capture traffic matching specific hosts/ports directly on an interface:

```
capture CAP1 interface INSIDE match ip host 192.168.200.200 host 192.168.100.100
```

This captures bidirectional traffic between the two hosts.

## Using an ACL Filter

For more complex filtering, define an ACL and reference it:

```
access-list BUF-FILTER extended permit ip host 10.50.49.240 any4
access-list BUF-FILTER extended permit ip any4 host 10.50.251.226
```

```
capture CAP1 access-list BUF-FILTER buffer 2048 circular-buffer
```

| Parameter         | Description                                    |
|-------------------|------------------------------------------------|
| `access-list`     | ACL name to filter captured traffic            |
| `buffer`          | Buffer size in KB                              |
| `circular-buffer` | Overwrites oldest data when buffer is full     |

## Match Filter Syntax

```
capture <Name> interface <Interface> match tcp host <Source IP> host <Destination IP> eq <Port>
```

## Viewing Captures

**CLI output:**

```
show capture CAP1
```

## Exporting Captures

**Download via ASDM (browser):**

```
https://<ASA-Management-IP>/admin/capture/CAP1/pcap
```

**Export via FTP from CLI:**

```
copy /pcap capture:CAP1 ftp://user:pass@192.168.1.1/CAP1.pcap
```

## Reading TCP Capture Output

The ASA displays TCP flows in this format:

```
HH:MM:SS.ms [ether-hdr] src-addr.src-port dest-addr.dst-port: tcp-flags [header-check] [checksum-info] sequence-number ack-number tcp-window urgent-info tcp-options
```

### Example: HTTP Request (TCP 3-Way Handshake + Data)

Client `1.1.1.1` connects to web server `2.2.2.2` on port 80.

**Packet 1 - SYN (client initiates):**

```
15:01:45.052762 1.1.1.1.12869 > 2.2.2.2.80: S 3624439037:3624439037(0) win 8192
```

The `S` flag indicates a SYN. The client is opening a connection to port 80.

**Packet 2 - SYN-ACK (server responds):**

```
15:01:45.053403 2.2.2.2.80 > 1.1.1.1.12869: S 285283040:285283040(0) ack 3624439038 win 8192
```

Both `S` (SYN) and `ack` are present. The source port 80 confirms this is return traffic from the web server.

**Packet 3 - ACK (handshake complete):**

```
15:01:45.054501 1.1.1.1.12869 > 2.2.2.2.80: . ack 285283041 win 260
```

The `.` with an `ack` completes the three-way handshake. If you see this, the connection is established.

**Packet 4 - PSH (client sends data):**

```
15:01:45.054852 1.1.1.1.12869 > 2.2.2.2.80: P 3624439038:3624439328(290) ack 285283041 win 260
```

The `P` (PUSH) flag means the client is sending application data (e.g., HTTP GET).

**Packet 5 - ACK (server acknowledges):**

```
15:01:45.244463 2.2.2.2.80 > 1.1.1.1.12869: . ack 3624439328 win 260
```

Server acknowledges receipt of the client's data.

**Packets 6-7 - Data transfer (server sends response):**

```
15:01:46.344296 2.2.2.2.80 > 1.1.1.1.12869: . 285283041:285284301(1260) ack 3624439328 win 260
15:01:46.344418 2.2.2.2.80 > 1.1.1.1.12869: . 285284301:285285561(1260) ack 3624439328 win 260
```

The `.` flag with data bytes indicates the server is streaming response data back to the client.

## Cleanup

**Clear capture data (keep the listener active):**

```
clear capture CAP1
```

**Remove the capture entirely:**

```
no capture CAP1
```
