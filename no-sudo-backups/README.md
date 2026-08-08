# ZFS Backups with Minimal Permissions

With two TrueNAS systems, designate one as the initiator "A" and the other as the listener "B".

## Creating a service account on B

Create a new dataset to hold data from A.
* Readonly: OFF

Create a new local user to act as the service account for A.
* SSH Access: Checked
* Disable Password: Checked
* Public SSH Key: <copied from external system>
* Home Directory: <dataset created earlier>
* Sudo Commands: none

Change the owner of the A's dataset to A's service account.

## Push backups from A to B

### Configure B to receive backups from A

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
