# Packet Capture on Cisco IOS-XE (CSR/ISR/ASR)

> **Applies to:** Cisco IOS-XE platforms (CSR1000v, ISR 4000, ASR 1000, Catalyst 8000, etc.)
>
> **Introduced in:** IOS-XE Release 3.7 / IOS 15.2(4)S

> **Key behavior:** Incoming packets hit the capture **before** any ACL, NAT, or other processing. Outgoing packets hit the capture **last**, right before being placed on the wire.

## Overview

IOS-XE uses a simplified capture syntax compared to classic IOS EPC. The buffer, interface, and filter are all managed under a single `monitor capture` command hierarchy.

## Step-by-Step Configuration

### 1. Define the Capture and Attach to an Interface

```
monitor capture BUF interface GigabitEthernet1 both
```

### 2. (Optional) Apply a Traffic Filter

Define an ACL:

```
ip access-list extended BUF-FILTER
  permit ip host 1.1.1.1 any
  permit ip any  host 1.1.1.1
```

Associate the filter with the capture. You can reference an ACL, a class-map, or specify the filter inline:

```
monitor capture BUF access-list BUF-FILTER buffer circular
```

### 3. Start the Capture

```
monitor capture BUF start
```

### 4. Stop the Capture

```
monitor capture BUF stop
```

## Viewing Captured Data

**Summary view:**

```
show monitor capture BUF buffer brief
```

**Detailed view:**

```
show monitor capture BUF buffer detailed
```

## Exporting Captures

Export as PCAP for analysis in Wireshark:

```
monitor capture BUF export ftp://10.0.0.1/CAP.pcap
```

Supported export destinations include FTP, TFTP, SCP, and local flash.

## Cleanup

Remove the capture when done:

```
no monitor capture BUF
```
