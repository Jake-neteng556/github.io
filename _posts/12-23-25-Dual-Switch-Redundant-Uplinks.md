# 2025-12-23 – Design Change: Dual Data Switch Redundant Uplinks (No Port-Channel Quite Yet)

## Summary
- Implemented redundant uplink design using two data switches:
  - Edge switch: Edge 24 PoE (PIET 250W)
  - Core/Access switch: Cisco CBS350
- Both switches have direct uplinks to the gateway router.
- Switch-to-switch link provides a failover path if either switch loses its direct uplink.
- Port-channeling between switches is planned but not yet implemented.

## Topology
- Gateway Router
  - Uplink A -> Edge 24 PoE (PIET 250W)
  - Uplink B -> Cisco CBS350
- Inter-switch link:
  - Edge 24 PoE (PIET 250W) <-> Cisco CBS350

## Intended Behavior (Redundancy)
- Normal operation:
  - Edge switch uses its direct gateway uplink.
  - CBS350 uses its direct gateway uplink.
- Failure scenario:
  - If the Edge switch loses its gateway uplink:
    - Edge -> Inter-switch link -> CBS350 -> Gateway -> Internet
  - If CBS350 loses its gateway uplink:
    - CBS350 -> Inter-switch link -> Edge -> Gateway -> Internet

## Current State
- Dual direct links to the gateway router
- Single inter-switch backup path
- No port-channel configured

## Future Work
- Implement LACP port-channel between switches
- Validate STP behavior post-change

## Validation
- Verified direct uplinks from both switches
- Verified failover via inter-switch path

## Notes

- Relies on correct L2/L3 configuration and STP
- The Intel NUC ESXi server farm is directly connected to the CBS350-24P switch. Consequently, if the switch suffers a hardware failure, there is no redundancy, as the system relies on a single physical point of failure.
- To address this single point of failure, I would ideally implement multi-homing via redundant NICs across both switches. Unfortunately, this is not a viable option given the current configuration of my lab containers.   


