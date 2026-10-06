# Install Plan (reviewed before each step runs)

Each step needs approval before it runs. Each step records evidence in the build log.

## 1. Patch and resize
1. `sudo apt update && sudo apt upgrade` on `soc-endpoint-01`; record package count; power off.
2. UTM: RAM 1.6 GB to 4096 MB (record before/after from the settings panel). Power off `noc-monitoring` to free host memory.
3. Boot; verify `free -h`, agent 002 Active, clock vs Mac.

## 2. Install
1. `sudo apt install suricata` (Ubuntu archive, expected `1:8.0.3-1`); record the installed version.
2. Back up the shipped config: `sudo cp -p /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.orig-20261006`.

## 3. Configure
1. **Capture interface:** set af-packet to `enp0s1` explicitly; confirm it is the only non-loopback interface (`ip -br link`) and that the service starts on it.
2. **HOME_NET:** the sensor's own address only (so scans from the scanner VM count as external).
3. Rules: `sudo suricata-update` (Emerging Threats Open); record the rule count it reports.
4. Validate before starting: `sudo suricata -T -c /etc/suricata/suricata.yaml`; start only if it passes.
5. Confirm packets are captured on `enp0s1` (stats counters increase).
6. Confirm `eve.json` rotation exists (logrotate) so the 3.6 GB free disk is not exhausted.

## 4. Wazuh integration
1. Back up `/var/ossec/etc/ossec.conf` (`sudo cp -p`).
2. Add one `<localfile>` block: `json` format, `/var/log/suricata/eve.json`.
3. Restart the agent; confirm it is Active and that logcollector reports reading the file. No `chmod 777`: if the agent cannot read the file, fix group membership/ownership instead.

## 5. Tests
1. **Known-alert test:** `curl http://testmynids.org/uid/index.html` from the sensor. Pass = the **same event** (matched on `flow_id` and signature ID) appears in `eve.json` and in the Wazuh alert.
2. **Scan test:** from `ubuntu-mgmt`, the exact scope in [CLAUDE.md](../CLAUDE.md). Record whether Suricata alerts; no expectation is assumed.
3. **Exposure audit:** compare open ports found with each repo's documented baseline.

## Rollback
- Suricata: `sudo systemctl disable --now suricata`; `sudo apt remove suricata`; restore `suricata.yaml.orig-*` if kept.
- Wazuh agent: restore the `ossec.conf` backup; restart the agent.
- RAM: set back to the recorded previous value in UTM.
