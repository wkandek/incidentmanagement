### Post-Mortem MMELK Outage November 2022 - incident-151

#### Description

On 2022-11-08 at 15:35 UTC the elasticsearch server (MMELK) for the Mobile Marketing (MM) system  became inaccessible. SRE was notified by monitoring, an analysis showed showed that the disk for the short term ELK data had failed. A new disk was defined on an available external disk array, mounted and functionality restored at 22:47 UTC. Logging data for the last 7 days starting from 2022-11-01 11:34 UTC to 2022-11-08 15:37 UTC was lost.

The MMELK server has no Service Level Objective (SLO). Its main use is as a troubleshooting support tool. Customer support uses it to supply operational reports to three customers (cust533, cust 432 and cust342 with a combined revenue of < 3M USD/year), but the reports are not under any Service Level Agreement

#### Priority

Initially P1, but downgraded to P3 when it became clear no customer facing reports were pending.

#### Detection

Monitoring for the index rate on MMELK alerted SRE to a problem with the indices on 2022-11-08 at 15:38 UTC. There were other, low level alerts as well. At around the same time (15:39 UTC) users in customer support complained about a problem using MMELK through an internal Slack channel. SRE called incident-151.

#### Mitigation

A new disk was formatted on a disk array that had space available and the disk was mounted via fibre channel. When MMELK was originally set up the local SSD disk was put in place to guarantee best performance. We will need to monitor for performance problems due to the different disk architecture.

#### Root Cause

The disk for the short term ELK data had been implemented as a software array using 6x HP SSD drives. The HP SSD drives came with a firmware defect that locked up the disk once 32678 operating hours were reached, a bit over 3.5 years. All 6 disks had the same defect and came from the same batch, locking up around the same time. All data on the disk was lost.

HPE provides a firmware update to prevent the error from occurring. We did not install the firmware upgrade as we are not actively monitoring such updates.

[https://support.hpe.com/hpesc/public/docDisplay?docId=emr\_na-a00092491en\_us](https://www.google.com/url?q=https://support.hpe.com/hpesc/public/docDisplay?docId%3Demr_na-a00092491en_us&sa=D&source=editors&ust=1682184480247844&usg=AOvVaw310ulCQ9G9ZatgQcG0S_Eh)

#### Action Items

*   Audit other SSD drives for firmware issue - ticket OPS-3214: done 2022-11-09

*   2 drives found and fixed before 32678 hours over

*   Establish firmware update policy - ticket OPS-3216: done 2022-12-02

*   Firmware monitoring (disks, controllers, motherboard, dRAC/iLO, network cards) and installation falls to Infrastructure team and will be handled during normal quarterly patch window. A quarterly review for all devices is scheduled starting Q1 2023.

*   Review SLO for MMELK server - ticket OPS-3219: done - 2022-12-05

*   MMLEK needs to be 99% available, about 7 hours of downtime/month possible. This SLO requires a warm standby architecture. OPS-3304 filed for implementation.
