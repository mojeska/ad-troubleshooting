# repadmin

Shows and controls AD replication. Most problems are found with the first three commands.

## Is anything failing?

One line per DC: largest replication delta and failures, as source and as destination.
The quickest view of the whole forest:

```cmd
repadmin /replsummary
```

Every link with an error: which DC, which partner, which partition, and the error code:

```cmd
repadmin /showrepl * /errorsonly
```

Everything, to a CSV you can filter in Excel:

```cmd
repadmin /showrepl * /csv > C:\Temp\showrepl.csv
```

Inbound partners and last attempt for one DC:

```cmd
repadmin /showrepl DC01
```

## Forcing replication

Push every partition from DC01 to all its partners, across sites. Use after a fix to
confirm it works, or to get a change out quickly:

```cmd
repadmin /syncall DC01 /AdeP
```

`A` all partitions, `d` show names instead of GUIDs, `e` include other sites, `P` push.

Pull one partition from a specific partner:

```cmd
repadmin /replicate DC01 DC02 "DC=contoso,DC=com"
```

## Who changed what, and where

Attribute-level history for an object: which DC made each change, when, and the
version number. The answer to "who disabled this account and where did it start":

```cmd
repadmin /showobjmeta DC01 "CN=Jane Doe,OU=Users,DC=contoso,DC=com"
```

Changes waiting to replicate into a DC:

```cmd
repadmin /queue DC01
```

## Topology

Recalculate replication connections after site or site-link changes:

```cmd
repadmin /kcc *
```

Bridgehead servers per site, and the Inter-Site Topology Generator:

```cmd
repadmin /bridgeheads
repadmin /istg
```

## Backups and tombstone lifetime

Last backup per partition. If a DC has been offline longer than the tombstone lifetime
(usually 180 days), don't reconnect it. Demote it and clean up its metadata instead:

```cmd
repadmin /showbackup *
```

Check the tombstone lifetime:

```powershell
(Get-ADObject "CN=Directory Service,CN=Windows NT,CN=Services,$((Get-ADRootDSE).configurationNamingContext)" -Properties tombstoneLifetime).tombstoneLifetime
```

## Lingering objects

Objects that were deleted everywhere except one DC, usually after a long-offline DC
comes back. Check first with `/advisory_mode`, then run without it to remove them:

```cmd
repadmin /removelingeringobjects DC02 <DC01-DSA-GUID> "DC=contoso,DC=com" /advisory_mode
```

Get the DSA GUID of the good DC (`DC01`) from `repadmin /showrepl DC01` (the "DSA object GUID" line).

## Common errors

| Code | Meaning | Look at |
| --- | --- | --- |
| 1722 | RPC server unavailable | Firewall, DNS, DC down |
| 1256 | Remote system not available | Usually follows 1722 on the same link |
| 8524 | DSA operation unable to proceed because of a DNS lookup failure | `_msdcs` CNAME for the DC's GUID, DNS forwarders |
| 8453 | Replication access was denied | Permissions, or an RODC replicating a partition it shouldn't |
| 8606 | Insufficient attributes to create an object | Lingering objects |
| 8614 | Not replicated longer than tombstone lifetime | Long-offline DC. Don't force it back in |
| 1908 | Couldn't find the domain controller | DNS, DC locator |
| -2146893022 | Target principal name is incorrect | DC computer account password or secure channel, SPN |
