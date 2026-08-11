# ZFS Backups with Minimal Permissions

With two TrueNAS systems, designate one as the initiator "A" and the other as the listener "B".

## Create periodic snapshots on A and B

Create periodic snapshots for all datasets that should be backed up on the opposing system.

## Creating an SSH Connection

Create an SSH key pair on A.

Create a new dataset to hold data from A.
* Readonly: OFF

Create a new local user to act as the service account for A.
* SSH Access: Checked
* Disable Password: Checked
* Public SSH Key: \<copied from A>
* Home Directory: \<dataset created earlier>
* Sudo Commands: none

Change the owner of the A's dataset to A's service account.

Create an SSH connection on A with the address of B.

## Prepare B to receive backups from A

Required permissions to receive backups pushed by the external system:
* create - create datasets and child datasets
* destroy - cleanup outdated snapshots
* mount - transitive via create and receive commands
* readonly - set the readonly flag on created datasets
* receive - initiate inbound data transfer 
* snapshot - create snapshots

To set these permissions, run the following command in the web shell:
```bash
zfs allow <user-a> create,destroy,mount,readonly,receive,snapshot <dataset-a>
```

## Push backups from A

Create a new replication task
* Direction: PUSH
* Transport: SSH+NETCAT (assumes secure connection already exists)
* Use Sudo for ZFS Commands: Unchecked
* SSH Connection: \<created earlier>
* Netcat Active Side: REMOTE
* Netcat Active Side Min Port: >1024 (non-protected port)
* Netcat Active Side Max Port: >1024 (non-protected port) (may be same as Min port)
* Source: \<local dataset to send>
* Destination: \<dataset under service account>
* Destination Dataset Read-only Policy: SET
* Periodic Snapshot Tasks: \<created earlier>
* Run Automatically: Checked

## Prepare B to send backups to A

Required permissions to send backups pulled by the external system:
* send - initiate outbound data transfer; encrypted datasets are decrypted first
  * send:raw - encrypted datasets are not decrypted; unencrypted datasets sent unmodified
  * send:encrypted - encrypted datasets are not decrypted; unencrypted datasets are forbidden

To set these permissions, run the following command in the web shell:
```bash
zfs allow <user-a> send <dataset-b>
```
