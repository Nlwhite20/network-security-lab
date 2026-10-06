# Build Log

All times UTC. Real addresses are kept out of this file.

| Date / time | Change or check | Purpose | Verification | Result |
| --- | --- | --- | --- | --- |
| 2026-10-06 02:51 | Identified agent 002 (`soc-endpoint-01`) as the UTM VM named `Linux`; `ubuntu-mgmt` has no agent | Choose the sensor host | `hostname`, `dpkg-query wazuh-agent` on each VM (read-only) | Agent 4.14.7-1, service active |
| 2026-10-06 | Noted: kernel now `7.0.0-34`; agent reported `7.0.0-31` on 2026-09-28 | Change record | Manager `agent_control -i 002` vs `uname -r` | VM updated and rebooted between those dates without a log entry |
| 2026-10-06 02:57 | Agent 002 reconnected after the SIEM VM restart | Restore endpoint telemetry | `agent_control`: Active; fresh rootcheck alerts | Complete, no repair needed |
| 2026-10-06 13:55 | Deliberate test: one SSH connection as non-existent user `wazuh-test-invalid` | Prove a fresh event reaches the SIEM | Manager alert rule 5710 at 13:55:52.6, about 2 s after sending; agent reads sshd via journald | Complete. Two earlier runs (03:08, 03:14 VM time) also produced 5710 alerts |
| 2026-10-06 14:01 | Endpoint clock about 3 h 12 min slow; fixed with `chronyc burst 4/4` then `makestep` (approved sudo) | Accurate timestamps for IDS evidence | Endpoint, SIEM and Mac agree within 1 s | Complete |
| 2026-10-06 14:0x | Read-only preflight | Resource and package check before install | 2 CPU, **1.6 GiB RAM** (1.1 available), 3.6 GB disk free; `enp0s1` virtio_net only interface; Ubuntu archive `suricata 8.0.3`, `nmap 7.98`; 25 updates pending | RAM too small (decision: 4 GB) |
| 2026-10-06 | Decisions | Scope the build | RAM to 4 GB; Ubuntu archive Suricata 8.0.3 (no third-party repo); patch before install | Approved |
| — | Patch soc-endpoint-01 (25 updates) and reboot | Current base before install | — | Pending |
| — | RAM 1.6 GB to 4 GB in UTM; power off `noc-monitoring` | Headroom for Suricata | — | Pending |
| — | Install and configure Suricata | IDS | — | Pending |
| — | Forward `eve.json` to Wazuh | SIEM integration | — | Pending |
| — | Test alert proven in `eve.json` and Wazuh; scan test; exposure audit | Evidence | — | Pending |
