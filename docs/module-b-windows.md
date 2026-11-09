# Shanghai 2026 Module B Windows Infrastructure

Selected checks from my Windows Server 2025 practice for the Shanghai 2026 Module B scenario. These excerpts cover DNS, DHCP, and Active Directory provisioning. The values are transcribed from the Module B lab notes. Hostnames, addresses, and account identifiers have been anonymized or omitted.

| Area | Selected result |
| --- | --- |
| DNS | The primary and secondary DNS servers returned SOA answers with serial `29` for the same zone. |
| DHCP | The inspected IPv4 scope was in the `Active` state. |
| Ansible | The Active Directory practice play ended with `unreachable=0` and `failed=0`. |

## DNS zone version

The notes contain two `Resolve-DnsName` queries for the same SOA record, one against each DNS server. The table groups the selected fields by query target.

| Query target | Type | PrimaryServer | SerialNumber |
| --- | --- | --- | --- |
| Primary DNS | SOA | `<PRIMARY_DNS_HOST>` | 29 |
| Secondary DNS | SOA | `<PRIMARY_DNS_HOST>` | 29 |

Both servers answered with the same advertised zone version for this check. This is a point-in-time check of one zone. To repeat it, query each server with `Resolve-DnsName -Type SOA -Server` and compare `SerialNumber`. Query an A or PTR record separately when troubleshooting that record.

## DHCP scope state

Selected field from the `Get-DhcpServerv4Scope -ScopeId` output in the notes:

```text
ScopeId     State
<SCOPE_ID>  Active
```

This checks the server's scope state. A client-side verification would confirm a lease, default gateway, and DNS servers with `ipconfig /all`.

## Active Directory provisioning with Ansible

The practice play used Debian as the Ansible control node and WinRM to reach the domain controller. It used sample user data to exercise OU, security group, user, and home directory provisioning.

Selected fields from the recorded Play Recap:

```text
<DOMAIN_CONTROLLER> : ok=5 changed=4 unreachable=0 failed=0
```

Four tasks reported changes and the play reported zero failures for this run. The notes identify it as a practice run with sample data. Client sign-in and mapped-drive checks are separate validation steps.

## Command references

- [Microsoft Learn: Resolve-DnsName](https://learn.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname?view=windowsserver2025-ps)
- [Microsoft Learn: Get-DhcpServerv4Scope](https://learn.microsoft.com/en-us/powershell/module/dhcpserver/get-dhcpserverv4scope?view=windowsserver2025-ps)
- [Ansible documentation: microsoft.ad.user](https://docs.ansible.com/projects/ansible/latest/collections/microsoft/ad/user_module.html)
