# Sites and subnets

A client is matched to an AD site by its IP address. If its subnet isn't defined in
AD Sites and Services, the client has no site. It may log on against a DC in another
city and get DFS referrals to remote file servers. Users notice slow logons and
files that take forever to open.

## Clients with no site

Each DC logs them in `netlogon.log`. Find them:

```cmd
findstr /c:"NO_CLIENT_SITE" %SystemRoot%\debug\netlogon.log
```

From an admin workstation, across all DCs (the grouped version is in WindowsAdminToolkit as `Get-ADSiteSubnetGap`):

```powershell
Get-ADDomainController -Filter * | ForEach-Object {
    Select-String "\\$($_.HostName)\admin$\debug\netlogon.log" -Pattern 'NO_CLIENT_SITE'
}
```

## What's defined

```powershell
Get-ADReplicationSubnet -Filter * | Select-Object Name, Site | Sort-Object Name
Get-ADReplicationSite -Filter * | Select-Object Name
```

## Adding a subnet

```powershell
New-ADReplicationSubnet -Name '10.20.30.0/24' -Site 'Irvine' -Description 'Irvine office VLAN 30'
```

Let it replicate, then on a client in that subnet confirm it picks up the site:

```cmd
nltest /dsgetsite
nltest /dsgetdc:contoso.com /force
```

## DFS follows the same rule

DFS namespaces order referrals by site. Clients without a site, or a site-costing
setting that's off, get referrals to targets in other sites. See which target a path
resolves to, and the referrals this client has cached (needs the DFS Management tools):

```cmd
dfsutil diag viewdfspath \\contoso.com\files\shared
dfsutil cache referral
```

Clear the cached referrals after fixing subnets:

```cmd
dfsutil cache referral flush
```
