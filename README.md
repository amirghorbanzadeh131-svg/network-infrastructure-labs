# Enterprise Networking Lab in EVE-NG

Personal Cisco networking lab by **Amir Ghorbanzadeh**.

## Project overview

I use EVE-NG to practice implementing and troubleshooting switching, routing, and branch connectivity. This project presents selected operational results from my lab without publishing the underlying exercise or complete device configurations.

## What I implemented

- Link aggregation using LACP EtherChannel.
- IPv4 routing and route exchange using EIGRP.
- Branch connectivity using DMVPN with mGRE and NHRP.

## Verified results

| Area | Observed result |
| --- | --- |
| Switching | A two-member LACP EtherChannel was active on both inspected switches. The inspected links were already operating at full duplex. |
| Routing | EIGRP neighbors and learned routes were present on the inspected switching interconnect and tunnel. |
| Tunneling | The mGRE tunnel was up/up, DMVPN peers reached UP, and NHRP registration replies were recorded. |
| Dynamic branch connectivity | A dynamic remote-spoke mapping and a more-specific NHRP route were installed. A trace to the remote branch reached it in one overlay hop. |
| Reachability | Three targeted tunnel and inter-branch ICMP tests each returned 5 replies from 5 probes. |

The three ICMP tests covered a branch loopback to a hub tunnel address, a branch loopback to the other branch loopback, and a branch LAN router interface to the remote branch LAN router interface.

## Selected evidence

See [Selected verification results](docs/selected-verification.md) for anonymized, transcribed output excerpts and the scope of each check.

Original addresses, device identifiers, interface identifiers, topology, and source exercise materials are withheld. Excerpts are selected transcriptions, not unedited console screenshots.

## Validation scope

This is a personal lab project. The evidence demonstrates the selected runtime behavior and short reachability tests. IPsec encryption, failover, sustained performance, and complete scenario compliance were not validated.

