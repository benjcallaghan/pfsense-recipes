# ZFS Backups with Minimal Permissions

## Receiving backups from an external system

Required permissions to receive backups pushed by the external system:
* create - create datasets and child datasets
* destroy - cleanup outdated snapshots
* mount - transitive via create and receive commands
* readonly - set the readonly flag on created datasets
* receive - initiate inbound data transfer 
* snapshot - create snapshots

To set these permissions, run the following command in the web shell:
```bash
zfs allow <user> create,destroy,mount,readonly,receive,snapshot <dataset>
```
