# Kerberos and secure channel

## "The trust relationship between this workstation and the primary domain failed"

The computer account password is out of sync, usually after restoring a snapshot.
Check and repair without leaving the domain, run on the affected machine:

```powershell
Test-ComputerSecureChannel -Verbose
Test-ComputerSecureChannel -Repair -Credential contoso\admin
```

Or with nltest:

```cmd
nltest /sc_verify:contoso.com
nltest /sc_reset:contoso.com
```

## Tickets

Current user's Kerberos tickets, and clearing them after a group membership change
(log off and on also works):

```cmd
klist
klist purge
```

The computer account's tickets (e.g. after adding a server to a security group, to
avoid a reboot):

```cmd
klist -li 0x3e7 purge
```

## SPNs

Duplicate SPNs break Kerberos for that service and it falls back to NTLM, or fails.
Find duplicates in the forest:

```cmd
setspn -X -F
```

SPNs on an account, and searching for one:

```cmd
setspn -L svc-sql
setspn -Q MSSQLSvc/sql01.contoso.com*
```

Add one (`-S` checks for duplicates first, unlike the old `-A`):

```cmd
setspn -S HTTP/intranet.contoso.com contoso\svc-web
```

## Events

On DCs, Security log: **4768** (TGT requested), **4769** (service ticket), **4771** (pre-auth
failed, often a bad password or clock skew). Failure code `0x18` is a bad password,
`0x25` is clock skew.
