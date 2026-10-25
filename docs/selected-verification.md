# Selected verification results

Verification date: **25 October 2026**  
Environment: **Cisco virtual devices in EVE-NG**

These are selected, anonymized transcriptions of observed console output. Placeholders replace original addresses and identifiers. Columns and unrelated lines are omitted; repeated mappings are condensed. Status values and reported probe counts are retained.

## DMVPN peer state

Command: `show dmvpn`

Selected peer fields after convergence:

```text
Peer               State   Attribute
<HUB_PEER>         UP      S
<REMOTE_SPOKE>     UP      DT1
```

The remote-spoke entry was dynamic. Earlier observations during startup showed NHRP in the session-state column; the later observation shown here recorded UP.

## NHRP registration and dynamic mapping

Command: `show ip nhrp nhs detail`

Selected fields from an observed NHS row:

```text
NHS: <HUB_PEER>
Status flags: RE
Registration requests failed: 0
Registration replies received: 1
```

The command legend identifies R as responding and E as expecting replies. A separate `show ip nhrp` check showed a dynamic entry associated with the remote spoke and its branch network.

## Tunnel state and counters

Command: `show interfaces <TUNNEL>`

```text
<TUNNEL> is up, line protocol is up
Tunnel protocol/transport multi-GRE/IP
```

Two readings during the verification session showed:

| Counter | Earlier reading | Later reading |
| --- | ---: | ---: |
| Packets input | 70 | 123 |
| Packets output | 146 | 199 |

These counters include control traffic as well as test traffic. Their increase is not attributed exclusively to ICMP probes and is not a throughput measurement.

## Dynamic route selection

Command: `show ip route <REMOTE_BRANCH_LAN_ROUTER_IP>`

Before the branch traffic tests, the destination matched an EIGRP summary route through the tunnel. After dynamic NHRP resolution, a more-specific route was selected:

```text
Known via "nhrp", distance 250, metric 1
Next hop: <REMOTE_SPOKE_TUNNEL_IP>
Outgoing interface: <TUNNEL>
```

This does not imply that the broader EIGRP summary route was deleted.

A bounded traceroute to the remote branch recorded the remote spoke at overlay hop 1. This describes the overlay path, not the number of physical underlay hops.

## Targeted ICMP tests

| Source | Destination | Observed replies |
| --- | --- | --- |
| Branch loopback | Hub tunnel address | 5/5 |
| Branch loopback | Remote branch loopback | 5/5 |
| Branch LAN router interface | Remote branch LAN router interface | 5/5 |

Selected result line from each test:

```text
Success rate is 100 percent (5/5)
```

The last test terminates on a router LAN interface. It does not establish client-to-client application connectivity.

## LACP and EIGRP checks

Command: `show etherchannel summary`

Selected fields observed on each of the two inspected switches:

```text
Port-channel   Protocol   Members
<BUNDLE>(SU)   LACP      <LINK_A>(P) <LINK_B>(P)
```

S indicates Layer 2, U indicates in use, and P indicates a member bundled in the port-channel. Relevant inspected interfaces already showed full duplex.

An IPv4 EIGRP neighbor was observed on the switching interconnect. An EIGRP-learned route to the peer loopback was present, and a separate ICMP test to that loopback returned 5/5 replies. EIGRP neighbors and learned routes were also observed over the mGRE tunnel.

## Scope

These observations support the stated switching, routing, and branch-reachability results. They do not establish complete scenario compliance, encrypted transport, automatic failover, application availability, or production performance.

Full configurations, addressing plans, topology, credentials, and source exercise materials are not included.

## Command references

- [Cisco EIGRP command reference](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_eigrp/command/ire-cr-book/ire-s1.html)
- [Cisco EtherChannel configuration guide](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr2-fwd/etherchannel/etherchannel-configuration-guide/m_ethernetchannel.html)
- [Cisco DMVPN troubleshooting guide](https://www.cisco.com/c/en/us/support/docs/security/dynamic-multipoint-vpn-dmvpn/111976-dmvpn-troubleshoot-00.html)

