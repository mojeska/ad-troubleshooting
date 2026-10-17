# SYSVOL (DFSR)

SYSVOL holds Group Policy and logon scripts, and replicates between DCs with DFSR.
When GPOs behave differently depending on which DC a client hits, check here.

## Is SYSVOL on DFSR?

FRS was removed in Windows Server 2019. The state should be `Eliminated` (3) on every DC:

```cmd
dfsrmig /getglobalstate
dfsrmig /getmigrationstate
```

## Replication state on a DC

`4` is normal. `2` is initial sync, `5` means it's in error:

```powershell
Get-CimInstance -Namespace root\microsoftdfs -ClassName DfsrReplicatedFolderInfo |
    Select-Object ReplicatedFolderName, State
```

States: 0 Uninitialized, 1 Initialized, 2 Initial sync, 3 Auto recovery, 4 Normal, 5 In error.

## Backlog between two DCs

Files waiting to replicate from DC01 to DC02. A backlog that keeps growing means replication is stuck:

```cmd
dfsrdiag backlog /rgname:"Domain System Volume" /rfname:"SYSVOL Share" /smem:DC01 /rmem:DC02
```

PowerShell version (needs the DFS Management tools, `RSAT-DFS-Mgmt-Con`):

```powershell
Get-DfsrBacklog -GroupName 'Domain System Volume' -FolderName 'SYSVOL Share' `
    -SourceComputerName DC01 -DestinationComputerName DC02 -Verbose
```

Make a DC pick up configuration changes from AD now, instead of waiting for its poll:

```cmd
dfsrdiag pollad /member:DC01
```

## Events worth knowing

In the DFS Replication event log:

| Event | Meaning |
| --- | --- |
| 2213 | Replication paused after a dirty shutdown. The event text contains the exact command to resume it |
| 4012 | DC has been disconnected longer than `MaxOfflineTimeInDays` (default 60) and stopped replicating. Needs a non-authoritative sync |
| 4114 | Replicated folder disabled (expected during a restore) |
| 4602 | Authoritative sync initialized SYSVOL |
| 4614 / 4604 | Non-authoritative sync started / finished |

## Rebuilding SYSVOL on one DC (non-authoritative)

For a single DC with a bad or out-of-date SYSVOL. It throws away its copy and pulls a
fresh one from a partner. In ADSI Edit (Default naming context) open:

`CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<DC>,OU=Domain Controllers,DC=contoso,DC=com`

1. Set `msDFSR-Enabled` = `FALSE`. Force AD replication (`repadmin /syncall /AdeP`).
2. On that DC: `dfsrdiag pollad`. Wait for event **4114**.
3. Set `msDFSR-Enabled` = `TRUE`. Replicate AD again, then `dfsrdiag pollad`.
4. Wait for events **4614** and then **4604**. SYSVOL is rebuilt.

## Authoritative restore (all DCs)

Only when SYSVOL is wrong everywhere and one DC holds the copy you want to keep.
Follow Microsoft's procedure exactly:
[How to force authoritative and non-authoritative synchronization for DFSR-replicated SYSVOL](https://learn.microsoft.com/troubleshoot/windows-server/group-policy/force-authoritative-non-authoritative-synchronization).
In short: on the good DC set `msDFSR-Enabled` = `FALSE` and `msDFSR-Options` = `1`.
Set `msDFSR-Enabled` = `FALSE` on all the others. Replicate and wait for 4114 everywhere.
Re-enable the good DC first (event 4602), then the rest (4614, 4604).
