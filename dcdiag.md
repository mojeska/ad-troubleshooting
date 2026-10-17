# dcdiag

Runs a set of tests against one DC or all of them. Start with errors only. The full
output is long, and the failures are what matter.

## Health check

Errors only, every DC in the forest:

```cmd
dcdiag /e /q
```

One DC, verbose, saved to a file to read or send to someone:

```cmd
dcdiag /s:DC01 /v /f:C:\Temp\dcdiag-DC01.txt
```

Comprehensive run (adds tests not in the default set, like topology and cutoff servers):

```cmd
dcdiag /s:DC01 /c /v /f:C:\Temp\dcdiag-full.txt
```

## Individual tests

DNS: registration, forwarders, delegation and dynamic updates, for every DC.
The best single test when logons or replication point to name resolution:

```cmd
dcdiag /test:dns /e /v
```

Replication errors as dcdiag sees them (repadmin gives more detail):

```cmd
dcdiag /test:replications /e /q
```

Is the DC advertising itself as a DC, GC, time server and KDC? Fails when Netlogon
isn't running or SYSVOL isn't shared yet:

```cmd
dcdiag /s:DC01 /test:advertising /v
```

SYSVOL and NETLOGON shared and ready:

```cmd
dcdiag /s:DC01 /test:netlogons /test:sysvolcheck
```

Can every DC reach the FSMO role holders:

```cmd
dcdiag /test:fsmocheck
dcdiag /test:knowsofroleholders /e /q
```

Recent DFSR and system event errors on a DC:

```cmd
dcdiag /s:DC01 /test:dfsrevent /test:systemlog
```

## Notes

- `/e` tests every DC in the enterprise, so run it during business hours with care on large forests.
- A single failed test isn't always a problem. `SystemLog` often fails on old, already-fixed events. Read the actual error.
- Run `dcdiag /?` on the DC to see every test available for that OS version.
