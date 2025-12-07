---
title: "Proxmox: Root Filesystem Crisis & Recovery"
date: 2025-12-07
---

## Overview
Documenting a root filesystem space emergency on a Proxmox host caused by an aborted VM disk conversion workflow. Includes findings, root cause, and resolution.

## Issue Summary
- Root FS reached **98% usage**
- System at risk of instability
- Major disk consumer found in `/root` directory

## Root Cause Analysis
A failed VM export operation redirected a large raw storage stream into a file in `/root`, unintentionally consuming ~84GB.

## Resolution Steps
1. Identified oversized file using:
    ```bash
    du -xhm --max-depth=1 /root
    ```
2. Removed the erroneous file safely:
    ```bash
    rm -- '-'
    ```
3. Verified restored capacity:
    ```bash
    df -h /
    ```

## Outcome
- System returned to safe operating space (**5% usage**)
- No filesystem corruption observed
- Follow-up actions planned to improve environment resilience

## Follow-Up Work
- Improve Proxmox storage layout for large exports
- Fix nested hypervisor reachability constraints
