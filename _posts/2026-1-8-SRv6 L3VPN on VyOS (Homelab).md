- ## SRv6 L3VPN on VyOS (Homelab)
  date:: 2026-01-08
  tags:: [[homelab]] [[vyos]] [[srv6]]

- **What**  
  SRv6-enabled PE using VyOS.

- **Why**  
  Learn how SRv6 L3VPN works without vendor magic.

- **Setup**
  - IS-IS for underlay
  - BGP VPNv4/VPNv6 for overlay
  - One VRF (VRF1)
  - One SRv6 locator

- **Key Bits**
  - Loopback: `2001:db8:2:ffff::2/128`
  - SRv6 locator: `2001:db8:2:aaa::/64`
  - VRF RD: `10.0.0.2:101`
  - VRF RT: `65000:101`
  - Per-VRF SID: `101`

- **Status**
  Control plane up. Data plane testing next.

- **Notes**
  SRv6 in VyOS is powerful but easy to misconfigure.
