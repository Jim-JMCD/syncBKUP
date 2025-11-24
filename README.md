# UNDER CONTRUCTION 
## syncBKUP - Synchronised data backups with unlimited versioning. 

#### _SyncBKUP_ a Bash script to that synchronises data directories and files to a backup repository. Only the last backup is stored in the repository as a complete copy of the original.  All changes to a directory or file between backups are kept in individual version folders. The history of changes recorded version folders no limits.

### Usage
~~~
syncBKUP -s <source> -d <destination> -m <no-mount-point-check>
~~~
#### Options

**-s** Data source to be backed up can be either a single directory or a file listing directories to be backed up. 

**-d** The destination directory of the backup data, the backup repository.

**-m** When used it must be accompanied by "no-mount-point-check".   
