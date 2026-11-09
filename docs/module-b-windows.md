# Shanghai 2026 Module B Windows Infrastructure

Selected configuration and verification notes from my Windows Server 2025 practice for the Shanghai 2026 Module B scenario. They cover RRAS site-to-site VPN, Group Policy, DNS, DHCP, and Active Directory provisioning. Recorded values come from the Module B lab notes. Hostnames, addresses, and account identifiers have been anonymized or omitted.

| Area | Configuration or evidence |
| --- | --- |
| Site-to-site VPN | RRAS IKEv2 with demand-dial interfaces and routes to remote IPv4 networks. |
| Group Policy | The GPO editor shows the built-in Administrator account policy set to `Disabled`. |
| DNS | The primary and secondary DNS servers returned SOA answers with serial `29` for the same zone. |
| DHCP | The inspected IPv4 scope was in the `Active` state. |
| Ansible | The Active Directory practice play ended with `unreachable=0` and `failed=0`. |

## RRAS IKEv2 site-to-site VPN

The lab notes document two Windows Server edge routers with RRAS demand-dial interfaces using IKEv2. Each edge has routes for the other site's IPv4 networks. The paired routes are intended to carry forward and return traffic through the tunnel.

To verify the tunnel and routing on each edge, substitute the interface name and a destination at the opposite site:

```powershell
Get-VpnS2SInterface -Name '<RRAS_INTERFACE>' |
    Select-Object Name, Protocol, ConnectionState, IPv4Subnet

Find-NetRoute -RemoteIPAddress '<REMOTE_HOST_IP>'
```

The notes specify `Connected` as the expected state. For a future run, check that the interface reports IKEv2 and `Connected`, and that the selected route uses the site-to-site interface. Repeat the lookup on the other edge and test the required service from a client at each site.

## Group Policy local Administrator control

The Group Policy Management Editor screenshot in the notes shows this computer policy under **Windows Settings → Security Settings → Local Policies → Security Options**:

| Setting | Value shown |
| --- | --- |
| Accounts: Administrator account status | Disabled |

This policy controls the built-in local Administrator account on targeted domain members. The screenshot records the value in the GPO editor. To check the effective state, run these commands in an elevated 64-bit PowerShell session on a targeted member computer:

```powershell
gpresult /scope computer /r
Get-LocalUser | Where-Object { $_.SID.Value -match '-500

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

- [Microsoft Learn: Get-VpnS2SInterface](https://learn.microsoft.com/en-us/powershell/module/remoteaccess/get-vpns2sinterface?view=windowsserver2025-ps)
- [Microsoft Learn: Find-NetRoute](https://learn.microsoft.com/en-us/powershell/module/nettcpip/find-netroute?view=windowsserver2025-ps)
- [Microsoft Learn: Group Policy results](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-modeling-results)
- [Microsoft Learn: Accounts Administrator account status](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/accounts-administrator-account-status)
- [Microsoft Learn: Resolve-DnsName](https://learn.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname?view=windowsserver2025-ps)
- [Microsoft Learn: Get-DhcpServerv4Scope](https://learn.microsoft.com/en-us/powershell/module/dhcpserver/get-dhcpserverv4scope?view=windowsserver2025-ps)
- [Ansible documentation: microsoft.ad.user](https://docs.ansible.com/projects/ansible/latest/collections/microsoft/ad/user_module.html)
 } |
    Select-Object Name, Enabled
```

The expected result is the intended GPO in **Applied Group Policy Objects** and `Enabled=False` for the built-in local Administrator. Checking the SID also works when the account has been renamed.

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
