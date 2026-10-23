# AD troubleshooting notes

The commands I reach for when Active Directory misbehaves: replication, DNS and DC
locator, SYSVOL, time, FSMO, Kerberos and Group Policy. Each entry is a line or two
on what it's for, then the command. Replace `contoso.com`, `DC01` and friends with
your own names.

Run these from an elevated prompt on a DC, or from an admin workstation with RSAT.

| Notes | Use when |
| --- | --- |
| [dcdiag](dcdiag.md) | You need an overall health check, or a specific DC test |
| [repadmin](repadmin.md) | Replication is failing, slow, or a change isn't showing up |
| [DNS and DC locator](dc-locator-dns.md) | Clients find the wrong DC, logons are slow, "domain not available" |
| [Sites and subnets](sites-subnets.md) | Clients authenticate or get DFS referrals across the WAN |
| [SYSVOL (DFSR)](sysvol-dfsr.md) | GPOs differ between DCs, SYSVOL isn't replicating |
| [Time](time.md) | Kerberos errors, clock skew, PDC time source |
| [FSMO roles](fsmo.md) | Finding, transferring or seizing roles |
| [Kerberos and secure channel](kerberos-secure-channel.md) | Trust relationship failed, SPN problems, ticket issues |
| [Group Policy](group-policy.md) | A policy isn't applying |

## Disclaimer

> [!WARNING]
> **Use at your own risk.** These notes and the commands in them are provided as-is, with no warranty or guarantee of any kind. The author accepts no liability for any damage, data loss, downtime or other problems that result from using them.
>
> Read and understand anything before you run it, do your own research, and test in a lab before using it in production. See the [MIT License](LICENSE) for the full terms.

## First five minutes

When someone says "AD is broken", this is the order I check things in:

```powershell
dcdiag /e /q                         # errors only, every DC in the forest
repadmin /replsummary                # any DC failing to replicate, and for how long
repadmin /showrepl * /errorsonly     # which partner and partition, with the error
nltest /dsgetdc:contoso.com          # which DC this machine actually finds
w32tm /monitor                       # time offset on every DC
dfsrmig /getmigrationstate           # SYSVOL is on DFSR and all DCs agree
```

The PowerShell versions of several of these checks live in
[WindowsAdminToolkit](https://github.com/mojeska/WindowsAdminToolkit).
