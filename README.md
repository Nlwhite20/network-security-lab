# Network Security Lab: Suricata IDS + Wazuh

> Status: **built and tested (2026-10-06).** Suricata 8.0.3 runs on the sensor and feeds
> Wazuh. Known-alert test passed (same `flow_id` in `eve.json` and the Wazuh alert, about
> 45 ms apart). A scoped nmap scan raised 5 Suricata alerts, all 5 in Wazuh. Exposure audit
> found no undocumented open ports and independently confirmed the Lab 2 dashboard fix.

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
- [Scan test and exposure audit](docs/scan-and-exposure-audit.md)
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
| Install Suricata | A plan on the existing 1.6 GiB VM | Required a resource and package preflight first | Memory too small; raised to 4 GB before install. Suricata then used about 1.2 GiB |
| Patch the sensor | `apt upgrade` | Held `wazuh-agent` first | Prevented the agent upgrading to 4.14.8, past the 4.14.7 manager |
| Find the Suricata test alert in Wazuh | Search alerts for the number `2100498` | Checked the matched alert's rule | It matched a sudo alert whose ID contained those digits; search now matches `signature_id` exactly |
| Known-alert test | Use `testmynids.org` | Checked DNS from three places | Site did not resolve; replaced with an in-lab test using the same signature and no config change |
| Scan both targets | One unprivileged nmap command for both hosts | Checked that each target had a scan report | The SIEM had none (firewall dropped host discovery); rescanned it with `-Pn` after approval |
| Run the scan | A scan command that printed the scanner clock but did not check it | Compared it with the Mac afterwards | Scanner was about 3 h 10 min slow; Run 1 timestamps marked unreliable, clock fixed before Run 2 |

Limitations: single-operator lab, so review is not independent.
