# UNDER CONTRUCTION 
## syncBKUP - Synchronised data backups with unlimited versioning. 

#### _SyncBKUP_ a Bash script to that synchronises data directories and files to a backup repository. Only the last backup is stored in the repository as a complete copy of the original.  All changes to a directory or file between backups are kept in individual version folders. The history of changes recorded version folders no limits.

### Usage
~~~
syncBKUP -s <source> -d <destination> -m <no-mount-point-check>
~~~
#### Options

**-s** Data source to be backed up can be either a single directory on the cammnd line or a file listing directories to be backed up. 

**-d** The destination directory of the backup data, the backup repository.

**-m** Optional, when used it must be accompanied by "**no-mount-point-check**".   

**Input restrictions**
* All directory paths must be <ins>full paths</ins>.
* <ins>All symbolic links</ins> are ignored and will not be processed.
* Single directory input with spaces require single quotes **'/mnt/c/Users/ted/My Documents'**    

**Source File Format**

A source file provides a list of directtories to back up. The format requirments are 
* Any line that starts with *'#'* is ignored.
* Any line that starts with *'.'* (fullstop) and there are no other charecters on the line, this terminates backup processing. Any directories listed after this line will not be processd.
* All blanks lines are ignored.
* Any line the starts with vaild non-empty directory will be processed.
* Leading spaces on valid non-empty directory entries are permittedd 

  
