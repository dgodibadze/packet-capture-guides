# Cisco Packet Capture Reference

Step-by-step guides for capturing packets directly on Cisco network devices. No external span ports or tap devices required.

## What's Here

| Guide | Platform | Use Case |
|-------|----------|----------|
| [Cisco IOS EPC](docs/cisco-ios-epc.md) | IOS routers (non-XE) | Embedded Packet Capture using buffer + capture point workflow |
| [Cisco ASA](docs/cisco-asa.md) | ASA firewalls | Inline capture with ACL filters, includes TCP flag analysis |
| [Cisco IOS-XE](docs/cisco-ios-xe.md) | CSR1000v, ISR 4000, ASR 1000, Catalyst 8000 | Simplified EPC syntax introduced in IOS-XE 3.7 |

## When to Use These

You are troubleshooting a traffic flow and need to confirm what the device actually sees on the wire. Common scenarios:

- Verifying whether traffic arrives at or leaves an interface
- Confirming NAT translations are applied correctly (capture pre-NAT on ingress, post-NAT on egress)
- Isolating packet drops, resets, or retransmissions
- Validating ACL behavior by comparing captures on both sides of the policy

## Key Concept

All three platforms share the same capture behavior:

- **Ingress:** Capture fires **before** ACL, NAT, or any other processing
- **Egress:** Capture fires **after** all processing, right before the packet hits the wire

This matters when you are trying to determine whether a device is dropping, translating, or forwarding traffic. Capture placement relative to the processing pipeline tells you what the device is doing to the packet.

## Quick Reference

```
# IOS (classic)
monitor capture buffer BUF size 2048 max-size 1518 circular
monitor capture point ip cef POINT Ethernet0/0 both
monitor capture point associate POINT BUF
monitor capture point start POINT

# IOS-XE
monitor capture BUF interface GigabitEthernet1 both
monitor capture BUF start

# ASA
capture CAP1 interface INSIDE match ip host 192.168.200.200 host 192.168.100.100
```

## Contributing

If you find errors or want to add captures for other platforms (Nexus, IOS-XR, Meraki, etc.), open an issue or submit a PR.

## License

This repository is provided as-is for reference purposes.
