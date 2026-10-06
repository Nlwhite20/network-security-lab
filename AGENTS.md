# Network Security Lab Safety Rules

This repository supports an authorized personal home-lab project only.

## Authorized Scope

- Only virtual machines I own on my UTM lab network. No public systems, employer,
  school, neighbor, customer or third-party networks.
- **Scan scope (exact):** scanner `ubuntu-mgmt`; targets `soc-endpoint-01` and
  `wazuh-siem` only, each confirmed by hostname at run time. TCP connect scan
  (`nmap -sT -p- -T3 --reason`), no root, no scripts (`-sC`/`--script`), no OS or
  version probing, no UDP, no other addresses. Any change to this scope needs my approval first.
- Known-alert test stays inside the lab: the Mac serves a text file containing
  `uid=0(root)` on the UTM network address only (temporary server, port 8080), and the
  sensor fetches it once (signature 2100498). Changed 2026-10-06 (approved) because
  `testmynids.org` did not resolve. No other traffic is sent outside the lab except package downloads.

## Required Approval

Ask before:
- Running `sudo`, installing or upgrading software, or changing VM settings (RAM, CPU, network)
- Editing Suricata, Wazuh agent or firewall configuration
- Running any scan, or changing the scan scope
- Deleting files, logs, agents or VMs
- Committing, pushing, or sending data outside the local lab

## Configuration and Security

- Back up every config file before editing it (`cp -p file file.orig-YYYYMMDD`).
- Never use `chmod 777` or loosen permissions to make a log readable; fix ownership or groups instead.
- Do not commit passwords, enrollment keys, certificates, raw logs, pcaps or full `eve.json` files.
- No real IP addresses, MAC addresses or usernames in published docs; use `<LAB_SUBNET>`-style placeholders.
- Check the VM clock against the Mac before any timed test.
- Every change gets a row in `docs/build-log.md` the same day.
