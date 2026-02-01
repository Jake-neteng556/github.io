ZFS Pool Recovery – After Action Report

date:: 2026-02-01
tags:: [[zfs]] [[truenas]] [[homelab]] [[recovery]]

Summary

Primary ZFS pool became unrecoverable after an unexpected power loss. Pool metadata could not be safely imported. Recovery was completed using an existing replicated backup pool.

Impact

Primary pool unavailable

No data loss due to ZFS replication

Root Cause

Power loss during active writes

Hardware RAID controller write cache caused corrupted ZFS metadata

Recovery Actions

Abandoned failed import attempts on damaged pool

Created fresh destination pool

Restored data using zfs send | zfs recv from replicated backup

Verified integrity with zpool status

Corrected dataset mountpoints

Initiated full scrub

Result

Pool restored successfully

All datasets, jails, and shares recovered

No checksum or read/write errors

Lessons Learned

ZFS + RAID controller write cache is dangerous without battery-backed cache

Replication works — backups saved the day

Stop recovery attempts early when metadata corruption is confirmed

Follow-ups

Harden storage configuration

Review power protection and write cache policy

Continue regular ZFS replication