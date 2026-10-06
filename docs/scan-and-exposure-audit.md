# Scan Test and Exposure Audit (2026-10-06)

All times UTC. Addresses replaced with host names.

## Scope (from [CLAUDE.md](../CLAUDE.md))

| Item | Value |
| --- | --- |
| Scanner | `ubuntu-mgmt` (nmap 7.98, Ubuntu archive) |
| Targets | `soc-endpoint-01`, `wazuh-siem`, each confirmed by `hostname` over SSH immediately before scanning |
| Command | `nmap -sT -p- -T3 --reason` (unprivileged TCP connect, all 65,535 ports, no scripts, no OS/version probing, no UDP) |
| Approved change | `-Pn` added for `wazuh-siem` only (see Run 2) |

## Runs

| Run | Target | Duration | Result |
| --- | --- | --- | --- |
| 1 | `soc-endpoint-01` | 3 s | 22/tcp open; 65,534 closed (connection refused) |
| 1 | `wazuh-siem` | — | **Not scanned.** Unprivileged host discovery probes ports 80/443; the host firewall drops them, so nmap treated the host as down |
| 2 | `wazuh-siem` with `-Pn` | 106 s (14:33:10 to 14:34:56) | 22, 1514, 1515/tcp open; 65,532 filtered (no response) |

Run 1 timestamps on the scanner are not reliable: its clock was about 3 h 10 min slow and
the pre-test clock check was missed. The clock was corrected before Run 2. Suricata and Wazuh
times come from the sensor and SIEM, whose clocks had been checked.

nmap's service names for 1514/1515 (`fujitsu-dtcns`, `ifor-protocol`) are its default
port-table labels, not detected services; no version probing was done. These are the Wazuh
agent event and enrollment ports.

## Exposure audit: found vs documented

| Host | Port | Documented baseline | Found | Match |
| --- | --- | --- | --- | --- |
| `wazuh-siem` | 22 | Open (UFW allows OpenSSH) | Open | Yes |
| `wazuh-siem` | 1514, 1515 | Open on the lab network (accepted risk, wazuh-siem-lab) | Open | Yes |
| `wazuh-siem` | 443 dashboard | Loopback only (restored 2026-10-06, Finding 001) | Not reachable | Yes. Independent confirmation of the Finding 001 fix |
| `wazuh-siem` | 9200, 55000 | Loopback only | Not reachable | Yes |
| `wazuh-siem` | all others | UFW default deny | Filtered | Yes |
| `soc-endpoint-01` | 22 | SSH for administration | Open | Yes |
| `soc-endpoint-01` | all others | No baseline documented | Closed (refused, not filtered) | No mismatch, but no host firewall filtering was observed; host firewall state not checked |

**Result:** no undocumented open ports on either host.

## Detection of the scan (Run 1, against the sensor)

| Suricata signature (ET Open) | `eve.json` | Wazuh (rule 86601) |
| --- | --- | --- |
| ET SCAN Potential VNC Scan 5800-5820 | 1 | 1 |
| ET SCAN Suspicious inbound to MSSQL port 1433 | 1 | 1 |
| ET SCAN Suspicious inbound to mySQL port 3306 | 1 | 1 |
| ET SCAN Suspicious inbound to Oracle SQL port 1521 | 1 | 1 |
| ET SCAN Suspicious inbound to PostgreSQL port 5432 | 1 | 1 |

5 of 5 Suricata alerts reached Wazuh (later matched one-to-one on `flow_id`, signature ID,
direction and timestamp in wazuh-siem-lab
[Investigation 001](https://github.com/Nlwhite20/wazuh-siem-lab/blob/main/docs/investigation-001-lab3-ids-alerts.md),
which covers seven alerts in total: these five scan alerts plus two from the known-alert test). About one minute after the
scan, 14:32:01 to 14:32:25 UTC, the agent's event queue overflowed and reported that events may
be lost. The five matched alerts were received before then; any events lost in that window
cannot be listed, so this audit does not establish that collection was complete. A full TCP connect scan of 65,535 ports raised only
these five port-specific signatures; the default ET Open rules did not raise a generic
"port scan" alert. Run 2 (against the SIEM) was not visible to the sensor (see the
visibility limitation in the README).

## Raw evidence

nmap output files and install logs are kept on `ubuntu-mgmt` under `~/evidence/lab3/`, outside Git.
