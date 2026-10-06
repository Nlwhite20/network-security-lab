# Network Security Lab: Suricata IDS + Wazuh

> Status: **in progress.** Preflight complete; nothing installed yet.

## Purpose

Add network intrusion detection to the home lab and feed it into the existing
Wazuh SIEM ([wazuh-siem-lab](https://github.com/Nlwhite20/wazuh-siem-lab)), then audit
the lab's exposed ports against what each repo documents.

## Design

| Role | VM | Notes |
| --- | --- | --- |
| IDS sensor + Wazuh agent | `soc-endpoint-01` (Ubuntu 26.04 ARM64) | Suricata on `enp0s1`; Wazuh agent 002 reads `eve.json` |
| SIEM | `wazuh-siem` | Wazuh 4.14.7 single-node (Lab 2) |
| Scanner | `ubuntu-mgmt` | nmap, scope fixed in [CLAUDE.md](CLAUDE.md) |

**Limitation, stated up front:** UTM has no port mirroring. Suricata sees only
traffic to and from the sensor VM itself, not the whole lab network.

## Documentation

- [Build log](docs/build-log.md): every change, in order, with verification
- [Install plan](docs/install-plan.md): steps, backups, validation and rollback
- [Risk register](docs/risk-register.md)

## AI Collaboration and Human Validation

Built with Claude as an assistant. Claude proposed commands, plans and drafts;
I ran every command, approved every `sudo`, VM and config change, and every commit.
Rules: [CLAUDE.md](CLAUDE.md) / [AGENTS.md](AGENTS.md).

| What I asked for | What the AI proposed | What I checked or changed | Result |
| --- | --- | --- | --- |
| Find the VM behind Wazuh agent 002 | Assume it was `ubuntu-mgmt` (the earlier plan's first agent) | Checked each VM's hostname and installed packages | `ubuntu-mgmt` had no agent; it was the `Linux` VM |
| Identify the test alert | Match alerts by time | Matched by the unique test username instead | Found three runs of the test (not one), and that the endpoint clock was 3 h 12 min slow |
| Parse alert output | A `sed` pattern | Compared the parsed "rule 002" with the raw alert | The pattern captured the agent ID, not the rule ID; fixed |
| Install Suricata | A plan on the existing 1.6 GiB VM | Required a resource and package preflight first | Memory too small; raised to 4 GB before install |

Limitations: single-operator lab, so review is not independent.
