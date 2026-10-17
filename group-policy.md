# Group Policy

## Is it applying?

What applied to this computer and user, and what was filtered out and why. The HTML
report is easiest to read:

```cmd
gpresult /h C:\Temp\gpresult.html /f
gpresult /r /scope computer
```

Apply now:

```cmd
gpupdate /force
```

For a remote machine (opens a scheduled task there, needs remote admin):

```powershell
Invoke-GPUpdate -Computer PC123 -Force -RandomDelayInMinutes 0
```

## Common reasons a GPO doesn't apply

- Linked to the wrong OU, or the object is in a different OU than you think.
- Security filtering: the computer or user (or `Authenticated Users`, read) lacks Read and Apply.
- WMI filter evaluates false on that machine.
- Computer settings in a GPO linked to a user OU (or the reverse), without loopback.
- SYSVOL copy differs between DCs. Check [SYSVOL](sysvol-dfsr.md).

## Reports

All GPOs to HTML, for documentation or comparing before and after a change:

```powershell
Get-GPOReport -All -ReportType Html -Path C:\Temp\all-gpos.html
```

Which GPOs are linked where:

```powershell
Get-GPInheritance -Target "OU=Servers,DC=contoso,DC=com"
```

Back up every GPO before making changes:

```powershell
Backup-GPO -All -Path C:\GPOBackup
```
