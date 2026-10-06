# Risk Register

Status as of 2026-10-06.

| ID | Risk | Likelihood | Impact | Status | Treatment |
| --- | --- | --- | --- | --- | --- |
| R-01 | Sensor VM memory too small for Suricata (1.6 GiB) | High | Medium | Treatment approved | Raise to 4 GB before install |
| R-02 | 25 pending updates on the sensor | Medium | Medium | Treatment approved | Patch before install |
| R-03 | Ubuntu archive Suricata (8.0.3) is older than the latest upstream 8.0.x | Medium | Medium | Accepted | Chosen to avoid adding a third-party repo; re-review at next patch cycle |
| R-04 | VM clocks drift after the Mac sleeps (seen on all three lab VMs) | High | Medium | Open | Check clocks against the Mac before every timed test |
| R-05 | Changes made without a record (kernel update on the sensor between 09-28 and 10-06) | Occurred | Low | Open | Same-day build-log rule in CLAUDE.md |
| R-06 | IDS visibility limited to the sensor's own traffic (no port mirroring in UTM) | Certain | Medium | Accepted | Stated in README; tests are designed to involve the sensor |
| R-07 | `eve.json` growth fills the 3.6 GB free disk | Medium | Medium | Open | Verify log rotation before enabling |
