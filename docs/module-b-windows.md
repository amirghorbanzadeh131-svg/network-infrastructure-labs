# Shanghai 2026 Module B Windows Infrastructure

Selected configuration and results from my Windows Server 2025 practice. **Recorded Output** transcribes the lab notes. **Expected Output** shows the fields and values to look for when repeating a check. Environment identifiers are replaced with placeholders.

## RRAS IKEv2 site-to-site VPN

The notes document persistent IKEv2 demand-dial interfaces and routes to the opposite site's IPv4 networks on both RRAS edges.

### Tunnel state

```powershell
Get-VpnS2SInterface -Name '<RRAS_INTERFACE>' |
    Format-List Name, Protocol, ConnectionState, IPv4Subnet
```

**Expected Output** on each edge:

```text
Name            : <RRAS_INTERFACE>
Protocol        : Ikev2
ConnectionState : Connected
IPv4Subnet      : {<REMOTE_SUBNET_CIDR>:<METRIC>}
```

The notes specify `Connected` as the expected state. `IPv4Subnet` represents the remote route entries configured on that edge.

### Route selection

```powershell
Find-NetRoute -RemoteIPAddress '<REMOTE_HOST_IP>' |
    Where-Object { $_.DestinationPrefix } |
    Format-List DestinationPrefix, InterfaceAlias, NextHop
```

**Expected Output** for a destination covered by a remote subnet route:

```text
DestinationPrefix : <REMOTE_SUBNET_CIDR>
InterfaceAlias    : <RRAS_INTERFACE>
NextHop           : <NEXT_HOP>
```

The reverse lookup on the other edge should select its RRAS interface for the opposite subnet.

## Group Policy local Administrator control

**Recorded configuration** from the GPO Editor screenshot:

| Security Option | Value shown |
| --- | --- |
| Accounts: Administrator account status | `Disabled` |

### Applied computer policy

Run from elevated PowerShell on a targeted member computer:

```powershell
gpresult /scope computer /r
```

**Expected Output** for the relevant section of the report:

```text
Applied Group Policy Objects
----------------------------
    <COMPUTER_SECURITY_GPO>
```

### Built-in account state

```powershell
Get-LocalUser | Where-Object { $_.SID.Value -match '-500$' } |
    Select-Object Name, Enabled | Format-Table -AutoSize
```

**Expected Output** after the setting takes effect:

```text
Name                 Enabled
----                 -------
<BUILT_IN_ADMIN>       False
```

## DNS zone version

```powershell
Resolve-DnsName -Name '<AD_ZONE>' -Type SOA -Server '<PRIMARY_DNS_IP>'
Resolve-DnsName -Name '<AD_ZONE>' -Type SOA -Server '<SECONDARY_DNS_IP>'
```

**Recorded Output** (selected SOA fields from both responses):

| Query target | Record | Primary server | Serial |
| --- | --- | --- | --- |
| Primary DNS | SOA | `<PRIMARY_DNS_HOST>` | `29` |
| Secondary DNS | SOA | `<PRIMARY_DNS_HOST>` | `29` |

## DHCP scope state

```powershell
Get-DhcpServerv4Scope -ScopeId '<SCOPE_ID>' |
    Select-Object ScopeId, State | Format-Table -AutoSize
```

**Recorded Output** (selected scope fields):

```text
ScopeId     State
-------     -----
<SCOPE_ID>  Active
```

## Active Directory provisioning with Ansible

```bash
ansible-playbook -i inventory.yaml main.yml
```

**Recorded Output** from the Debian and WinRM practice run using sample user data:

```text
<DOMAIN_CONTROLLER> : ok=5 changed=4 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

## Command references

- [Microsoft Learn: Get-VpnS2SInterface](https://learn.microsoft.com/en-us/powershell/module/remoteaccess/get-vpns2sinterface?view=windowsserver2025-ps)
- [Microsoft Learn: Find-NetRoute](https://learn.microsoft.com/en-us/powershell/module/nettcpip/find-netroute?view=windowsserver2025-ps)
- [Microsoft Learn: gpresult](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/gpresult)
