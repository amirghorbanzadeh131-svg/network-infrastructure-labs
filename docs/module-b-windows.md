# Shanghai 2026 Module B Windows Infrastructure

Selected results from my Windows Server 2025 practice. The lab notes contain configuration evidence and recorded results. Host and scope identifiers are anonymized.

## RRAS IKEv2 site-to-site VPN

| Field | Documented value |
| --- | --- |
| VPN type | RRAS site-to-site |
| Protocol | IKEv2 |
| Interface | Persistent demand-dial |
| Routing | Remote IPv4 routes on both edges |
| Expected connection state | `Connected` |

## Group Policy local Administrator control

| Setting shown in GPO Editor | Recorded value |
| --- | --- |
| Accounts: Administrator account status | `Disabled` |

## DNS zone version

| Query target | Record | Primary server | Serial |
| --- | --- | --- | --- |
| Primary DNS | SOA | `<PRIMARY_DNS_HOST>` | `29` |
| Secondary DNS | SOA | `<PRIMARY_DNS_HOST>` | `29` |

## DHCP scope state

| Scope | Recorded state |
| --- | --- |
| `<SCOPE_ID>` | `Active` |

## Active Directory provisioning with Ansible

Play Recap from the Debian and WinRM practice run using sample user data:

```text
<DOMAIN_CONTROLLER> : ok=5 changed=4 unreachable=0 failed=0
```
