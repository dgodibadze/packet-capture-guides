# Cisco IOS Embedded Packet Capture (EPC)

> **Applies to:** Cisco IOS routers (non-XE platforms)

> **Key behavior:** Incoming packets hit the capture **before** any ACL, NAT, or other processing. Outgoing packets hit the capture **last**, right before being placed on the wire.

## Overview

Embedded Packet Capture (EPC) lets you capture traffic directly on a Cisco IOS router without external tools. You define a buffer, optionally apply a filter, attach it to an interface, and start capturing.

## Step-by-Step Configuration

### 1. Define the Capture Buffer

Create a temporary buffer in DRAM to store captured packets.

```
monitor capture buffer BUF size 2048 max-size 1518 circular
```

| Parameter   | Description                                      |
|-------------|--------------------------------------------------|
| `size`      | Buffer size in KB                                |
| `max-size`  | Maximum captured packet size in bytes            |
| `circular`  | Overwrites oldest data when buffer is full       |

### 2. (Optional) Apply a Traffic Filter

Define an ACL in config mode, then bind it to the buffer.

```
ip access-list extended BUF-FILTER
  permit ip host 150.1.0.4 any
  permit ip any  host 150.1.0.4
```

```
monitor capture buffer BUF filter access-list BUF-FILTER
```

### 3. Define the Capture Point

Specify where to capture, the protocol (IPv4/IPv6), and the switching path (process vs. CEF).

```
monitor capture point ip cef POINT Ethernet0/0 both
```

### 4. Attach the Buffer to the Capture Point

```
monitor capture point associate POINT BUF
```

### 5. Start the Capture

```
monitor capture point start POINT
```

### 6. Stop the Capture

```
monitor capture point stop POINT
```

## Viewing and Exporting Captured Data

**View hex dump on the router:**

```
show monitor capture buffer BUF dump
```

> This only shows a hex dump. To get human-readable output, either export as PCAP or copy the hex dump into an online hex-to-pcap converter.

**Export as PCAP via TFTP:**

```
monitor capture buffer BUF export tftp://10.10.163.36/BUF_JWTCRLV01.pcap
```

## Cleanup

Remove both the capture point and the buffer when done.

```
no monitor capture point ip cef POINT Ethernet0/0 both
no monitor capture buffer BUF
```

## Notes and Limitations

- The buffer is stored in DRAM. It does not survive a reload.
- The capture configuration is not stored in NVRAM. It does not survive a reload.
- On IOS releases earlier than 15.0(1)M, buffer size is limited to 512 KB and packet size to 1024 bytes.
- The capture point can target a specific interface or capture globally.
- The capture point supports both CEF and process switching paths.
- When exporting in PCAP format, Layer 2 information (e.g., Ethernet encapsulation) is **not** preserved.
