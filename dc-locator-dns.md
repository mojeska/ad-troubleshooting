# DNS and DC locator

Clients find DCs through DNS SRV records, then pick one in their own site. When
logons are slow or a client talks to a DC across the WAN, start here.

## Which DC does this machine use?

```cmd
nltest /dsgetdc:contoso.com
```

Shows the DC, its site, the client's site, and flags (GC, PDC, TIMESERV, KDC). Add
`/force` to ignore the cache and rediscover.

The DC that logged the current user on:

```cmd
echo %LOGONSERVER%
```

Which site this machine thinks it's in:

```cmd
nltest /dsgetsite
```

All DCs in the domain:

```cmd
nltest /dclist:contoso.com
```

PowerShell equivalent, with discovery in a specific site:

```powershell
Get-ADDomainController -Discover -DomainName contoso.com -SiteName Irvine
```

## Are the SRV records there?

All DCs in the domain, then just the ones in one site:

```cmd
nslookup -type=SRV _ldap._tcp.dc._msdcs.contoso.com
nslookup -type=SRV _ldap._tcp.Irvine._sites.dc._msdcs.contoso.com
```

Kerberos and the PDC:

```cmd
nslookup -type=SRV _kerberos._tcp.dc._msdcs.contoso.com
nslookup -type=SRV _ldap._tcp.pdc._msdcs.contoso.com
```

## Re-registering a DC's records

Netlogon registers the SRV records listed in `%SystemRoot%\System32\config\netlogon.dns`.
Restarting it re-registers them. The host A record comes from the DNS client:

```cmd
ipconfig /registerdns
net stop netlogon && net start netlogon
```

Check registration afterwards:

```cmd
dcdiag /test:registerindns /dnsdomain:contoso.com /v
```

## DNS settings on DCs

- Point each DC at another DC first and itself second (not `127.0.0.1` first). That avoids an island after a restore.
- No public DNS servers in a DC's NIC settings. Put them in forwarders.
- Check forwarders and root hints: `dcdiag /test:dns /e /v`.
