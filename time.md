# Time

Kerberos allows 5 minutes of clock difference by default. Beyond that, authentication
fails. Domain members sync from DCs, DCs sync from the PDC emulator of the forest
root domain, and only that PDC should sync from an outside source.

## Check

Offset of every DC from this one:

```cmd
w32tm /monitor
```

Where this machine gets its time, and when it last synced:

```cmd
w32tm /query /source
w32tm /query /status
```

Live offset against a specific server:

```cmd
w32tm /stripchart /computer:DC01 /samples:5 /dataonly
```

## PDC emulator: sync from an external source

Run on the forest-root PDC only:

```cmd
w32tm /config /manualpeerlist:"time.windows.com,0x8 pool.ntp.org,0x8" /syncfromflags:manual /reliable:yes /update
w32tm /resync /rediscover
```

## Every other DC and member: follow the domain hierarchy

```cmd
w32tm /config /syncfromflags:domhier /update
net stop w32time && net start w32time
w32tm /resync /rediscover
```

## Virtual DCs

Turn off host time sync for DC VMs (Hyper-V "Time synchronization" integration service,
or VMware Tools time sync). Otherwise the host and the domain fight over the clock.
After moving the PDC role, reconfigure the new PDC as above and set the old one back to `domhier`.
