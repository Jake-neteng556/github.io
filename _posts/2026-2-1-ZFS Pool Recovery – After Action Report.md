ZFS Pool Recovery – After Action Report

Date: 2026-02-01
Tags: ZFS, TrueNAS, homelab, recovery

Summary::

A primary ZFS pool became unrecoverable following an unexpected power loss. Pool metadata could not be safely imported. Recovery was completed using an existing replicated backup pool.

Impact::

Primary pool unavailable

No data loss due to prior ZFS replication

Root Cause::

Sudden power loss during active writes

RAID controller write cache caused ZFS metadata corruption

Recovery Actions::

Stopped further attempts to import the damaged pool

Created a new destination ZFS pool

Restored data using zfs send | zfs recv from replicated backups

Verified pool health with zpool status

Corrected dataset mountpoints

Initiated full ZFS scrub

Result::

Pool restored successfully

All datasets and services recovered

No read, write, or checksum errors detected

Lessons Learned::

ZFS + RAID write cache is unsafe without a battery-backed cache

Replication is critical, and the backups worked exactly as intended

Stop early when ZFS metadata corruption is confirmed

Follow-Up Actions::

Hardened storage and cache configuration

Review the power protection strategy

Continue regular ZFS replication

