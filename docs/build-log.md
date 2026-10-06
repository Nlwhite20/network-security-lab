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
| 2026-10-06 14:08 | `apt-mark hold wazuh-agent`, then patched (24 of 25 updates), powered off | Current base; keep agent at manager version | `wazuh-agent 4.14.8` was offered and would have run ahead of the 4.14.7 manager; held at 4.14.7. `upgrade exit=0`; reboot required | Complete |
| 2026-10-06 14:1x | UTM RAM raised to 4096 MB (previous value: to be confirmed from UTM); `noc-monitoring` already powered off | Headroom for Suricata | `free -h`: 3.3 GiB total, 2.8 GiB available; kernel `7.0.0-38`; agent 002 Active; clocks agree | Complete |
| 2026-10-06 14:14 | `apt install suricata` (Ubuntu archive) | IDS | `suricata 1:8.0.3-1`, `suricata-update 1.3.7-2`, logrotate config present. Service auto-started and **failed**: `af-packet: eth0: failed to find interface: No such device` | Installed; stopped until configured |
| 2026-10-06 14:14 | Backup `suricata.yaml.orig-20261006` | Rollback | `ls -l` | Complete |
| 2026-10-06 14:17 | Edited two lines only: af-packet interface `eth0` to `enp0s1` (line 660); `HOME_NET` to the sensor's own /32 (line 18) | Explicit capture interface; scanner traffic counts as external | Exact-match guards before `sed`; `diff` vs backup shows only lines 18 and 660 | Complete |
| 2026-10-06 14:17 | `suricata-update` (ET Open) | Detection rules | 69,065 rules, 53,110 enabled | Complete |
| 2026-10-06 14:17 | `suricata -T` passed, then service started | Validate before start | "Configuration provided was successfully loaded"; engine started on `enp0s1`, 2 worker threads; Suricata uses about 1.2 GiB (1.5 GiB still available) | Complete |
| 2026-10-06 14:19 | Capture check | Prove the interface sees traffic | `capture.kernel_packets` 137 to 890 during 10 pings to the gateway | Complete |
| 2026-10-06 14:20 | Backup `ossec.conf.orig-20261006`; added one `<localfile>` (json, `/var/log/suricata/eve.json`); restarted agent | SIEM integration | `wazuh-logcollector: Analyzing file: '/var/log/suricata/eve.json'`; agent Active; `eve.json` is 644, no permission change needed | Complete |
| 2026-10-06 14:25 | First known-alert attempt (`testmynids.org`) | Test | `curl: (6) Could not resolve host` from the sensor, the Mac and a separate network; Ubuntu archive reachable | Not run; replaced with an in-lab test (scope change approved) |
| 2026-10-06 14:28 | **In-lab known-alert test:** Mac served `uid=0(root)` text on the UTM address; sensor fetched it once | Prove the same alert in `eve.json` and Wazuh | `eve.json` 14:28:29.550573, signature 2100498, `flow_id` 1478072356148979. Wazuh alert 14:28:29.595, rule **86601**, agent 002, same `flow_id` | **Pass** (about 45 ms from sensor log to SIEM alert) |
| 2026-10-06 14:30 | Installed `nmap 7.98` on `ubuntu-mgmt`; scan Run 1 (targets confirmed by hostname first) | Scan test | Sensor: 22 open only. SIEM not scanned (host discovery dropped by firewall). 5 ET SCAN alerts in `eve.json`, same 5 in Wazuh | Partial. Scanner clock found about 3 h 10 min slow (pre-test clock check missed) |
| 2026-10-06 14:33 | Scanner clock fixed (`chronyc burst` + `makestep`, approved sudo); Run 2 on SIEM with `-Pn` (approved scope change) | Complete the exposure audit | SIEM: 22, 1514, 1515 open; 443/9200/55000 not reachable; 106 s | Complete. See [scan and exposure audit](scan-and-exposure-audit.md) |
