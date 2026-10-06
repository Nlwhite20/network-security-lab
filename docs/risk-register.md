# Risk Register

Status as of 2026-10-06.

| ID | Risk | Likelihood | Impact | Status | Treatment |
| --- | --- | --- | --- | --- | --- |
| R-01 | Sensor VM memory too small for Suricata (1.6 GiB) | High | Medium | Mitigated 2026-10-06 | Raised to 4 GB; Suricata uses about 1.2 GiB |
| R-02 | 25 pending updates on the sensor | Medium | Medium | Mitigated 2026-10-06 | 24 applied; `wazuh-agent` held at 4.14.7 to match the manager (upgrade both together later) |
| R-03 | Ubuntu archive Suricata (8.0.3) is older than the latest upstream 8.0.x | Medium | Medium | Accepted | Chosen to avoid adding a third-party repo; re-review at next patch cycle |
| R-04 | VM clocks drift after the Mac sleeps (seen on all lab VMs, including the scanner on 2026-10-06) | High | Medium | Open | Check clocks against the Mac before every timed test, and make the check stop the test; consider a chrony `makestep` setting on every VM |
| R-05 | Changes made without a record (kernel update on the sensor between 09-28 and 10-06) | Occurred | Low | Open | Same-day build-log rule in CLAUDE.md |
| R-06 | IDS visibility limited to the sensor's own traffic (no port mirroring in UTM) | Certain | Medium | Accepted | Stated in README; tests are designed to involve the sensor |
| R-07 | `eve.json` growth fills the 3.6 GB free disk | Medium | Medium | Open | Verify log rotation before enabling |
| R-08 | `HOME_NET` is a fixed /32; if DHCP gives the sensor a new address, detection direction breaks silently | Low | Medium | Open | Check the sensor address before each test; consider a DHCP reservation |
| R-09 | External IDS test site unavailable | Occurred | Low | Mitigated | In-lab test using the same signature |
| R-10 | Default ET Open rules did not flag a full TCP connect scan as a port scan (only 5 port-specific alerts) | Certain | Medium | Open | Consider enabling threshold-based scan detection or a Wazuh frequency rule on rule 86601 from one source |
| R-11 | Sensor answers closed ports with resets (no host firewall filtering observed) | Medium | Low | Open | Check UFW state on the sensor; decide on a baseline |
