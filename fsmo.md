# FSMO roles

Five roles. Losing one isn't an emergency except the PDC emulator, which handles time,
password changes and account lockouts, so users notice quickly.

## Who holds them

```cmd
netdom query fsmo
```

```powershell
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster
```

## Transfer (old holder is online)

The normal way, e.g. before decommissioning a DC:

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity DC02 -OperationMasterRole PDCEmulator, RIDMaster, InfrastructureMaster
```

Forest roles:

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity DC02 -OperationMasterRole SchemaMaster, DomainNamingMaster
```

## Seize (old holder is gone for good)

Same command with `-Force`. Only when the old DC will **never** come back online. A
seized-from DC must not return to the network. Clean up its metadata afterwards
(delete its computer object in AD Sites and Services or AD Users and Computers):

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity DC02 -OperationMasterRole PDCEmulator -Force
```

After moving the PDC emulator, set up time on the new one. See [Time](time.md).
