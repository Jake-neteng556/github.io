- ## SRv6 L3VPN on VyOS (EVE-NG)
  date:: 2026-01-08
  tags:: [[homelab]] [[vyos]] [[srv6]]

- **What**  
  SRv6-enabled PE using VyOS.

- **Why**  
  Discover how SRv6 L3VPN operates without vendor-specific magic.

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
  SRv6 in VyOS is powerful for service providers. 


<img width="2088" height="498" alt="image" src="https://github.com/user-attachments/assets/6a13cb99-5888-46de-b66a-e5cde278d0ca" />
