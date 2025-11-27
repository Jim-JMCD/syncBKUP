## syncBKUP - Synchronised data backups with unlimited delta versioning. 

#### _SyncBKUP_, a Bash script to that synchronises data directories and files to a backup repository. Only the most recent backup is stored in the repository. All changes and deletion to any directory or file between backups are kept in individual version directories. There are no limits on how long or how much historical data is kept, its user managed.  This script is based on the capabilities of ***rsync***.

### Usage
~~~
syncBKUP -s <source> -d <destination> -m <no-mount-check>
~~~
#### Options

**-s** The source data of the backup, the source must be a directory or the name of a file that contains directories to be backed up. 

**-d** The destination directory of the backup data, the backup repository. The repository contains the backed up data, logs and the delta versions.

**-m** An option that disables the mount point check for the backup target storage. When used it must be accompanied by "**no-mount-check**".   

**Input notes and restrictions** for all backups
* Only <ins>full directory paths</ins> are permitted. 
* <ins>All symbolic links</ins> are ignored and will not be processed.
* Directory input with spaces requires single quotes **'/mnt/c/Users/ted/My Stuff'**
* Trailing slashes on directory names are ignored.
* Individual files cannot be used as a source.    

**Source File Format** A source file that provides a list of directories to back up. 

The format requirements of the source file are: 
* Any line that starts with *'#'* is ignored.
* Any line where the first character is *'.'* (full stop) will terminate all backup processing.
* Blanks lines and leading spaces on lines are permitted.
* Only lines that contain a single valid non-empty directory will be processed, <ins>any additional material on a line will cause the line not to be processed</ins>. 
   
**Mount Point Check** A safety check to prevent the potential of the root and other filesystems filling up.

Traditionally */mnt* is where temporary external devices and network shares are attached. Prior to  attaching  (aka mounting) a device a directory (aka mount point) must exist. Anything that copies data to that mount point directory without an attached device will run as normal and <ins>could fill up the filesystem</ins>. When the mount point has an attached storage device, any data that was previously written to it remains and it is hidden and using up storage capacity.

* The scope of mount point check is the first three directories of the destination path, example **-d /mnt/USB/SG2TB/bkups/2025** the directories **/mnt**, **/mnt/USB** and **/mnt/USB/SG2TB** will be checked. The mount point check will fail if none of those directories contain a mounted device and backups will not proceed.
* Symbolic links to mount points are often used to create a convenient reference to storage devices. If the symbolic link is straight forward, it will pass the mount point check if it has an attached storage device. With symbolic links there are no guarantees, it pays to manually verify what is mounted.
* Manual verification is always best, run the **df -Th** command.
* Special case: In a MSYS2 environment the mount point check is disabled.
   
_Disabling Mount Point Check_ : The option **-m** must be accompanied with _no-mount-check_ : **-m no_mount_check**

### Data Recovery
The backup repository contains the most recent full backup in its unmodified native format, any file or directory copying utility can be used to recover data.  The same method can be be used to recover historical data from the version directories. See **RSYNC- Options used** section below and the internet for more information on which attributes are preserved in copying data.

~~~
rsync -aAXhv <source> <destination> 
~~~
Note: Trailing slashes at the end of the source path indicates that only the contents of the directory will be copied, omitting the slash means the entire directory, including its name, will be copied. 

### Backup Destination 

Directory structure of the backup repository:
 
* Level 1 - The directory given to syncBKUP as thre backup destination **-d \<destination\>**.  
* Level 2 - The computer name where the script was run, this identifies the source computer.
* Level 3 - A directory name derived from the source date directory provided by **-s \<source\>**, it only contains the Level 4 directory.  
* Level 4 - The directory level named after the parent of the data. Example if the source directory is **-s /home/ada/music** the name of this level will be **music**. This level contains: 
   * The synced backup data.
   * The log file
   * The versions directories. They contain all the deletions and modifications between each back.   

 The top level (Level 1) has to be a pre-existing directory. SyncBKUP will automatically create directories for Levels 2 to 4 when required. 
 
<ins>How to identify and locate what is backed up</ins> 

* Level 2 - Computer name of data source  
* Level 3 - Directory name is a modified form of the directory path of the source given to syncBKUP. The "/" slashes of the source directory path are replaced with underscores "_", spaces in directory names are preserved.

_Examples_
<!All backups can be located in **../Computer name/Modified_source_directory_name/..* -->
~~~
Computer name = star03
Data source   = -s /home/ada/music/2010s/dubstep
Destination   = -d /mnt/USB/BKUP_2TB/syncBKUP/2025

Location of the backup:
.../syncBKUP/2025/star03/home_ada_music_2010s_dubstep/dubstep/<backup data>
~~~
* Backups identifiers : **../star03/home_ada_music_2010s_dubstep/..**
* The log and version directories for backups are located in **.../dubstep/**

~~~
Computer name = fred02
Data source   = -s '/mnt/c/Users/ted/Music/1990 - 97/Shoegaze and Nu Metal'
Destination   = -d /mnt/e/syncBKUP/

Location of the backup:
'.../syncBKUP/fred02/mnt_c_Users_ted_Music_1900 - 97_Shoegaze & Nu Metal/Shoegaze & Nu Metal/<backup data>'
~~~
* Backups identifiers : **../fred02/mnt_c_Users_ted_Music_1900 - 97_Shoegaze & Nu Metal/..**
* Logs and version directories are located in **.../Shoegaze and Nu Metal/**

### Version Directories 
* For each directory backup, all file and directory deletions, modifications and renaming be recorded individual version directories.
* If there are no changes to a directory between backups a version directory is not created.
* When a file or directory is moved to the version directory its directory location within the original backup is also created. 
* If a source directory is renamed the entire contents of original directroy will be copied to the version file.
* All version directories have names that include the date and time of the backup that created them, format YYYY-MMM-DD-hhmm-ss.

Example of version directories 
~~~
Computer name    = star03
Data source      = -s /home/ada/
Destination      = -d /mnt/USB/BKUP_2TB/syncBKUP/2025
Backup path = .../syncBKUP/2025/star03/home_ada/ada/

Directories created by syncBKUP for the backup of star03:/home/ada/
.../star03/home_ada
              |__ /ada        <------------------------- Contains the most recent synchronised full backup of /home/ada 
              |__ /ada-version-2025-Nov-23-1902-16       
              |__ /ada-version-2025-Nov-09-1900-10       Version directories containing all the modificatons and deketions  
              |__ /ada-version-2025-Nov-02-1901-36       from each backup run (every Sunday at about 7PM)
              |__ /ada-version-2025-Oct-26-1900-05
              |__ /log        <------------------------  The log file for every /home/ada backup, all backup log data
                                                         is appended to this file
~~~

### Logs
* Individual logs are produced for each directory backup. Each backup log appends to the log file. 
* If the log file is deleted a new log file will be automatically created.
* Logs are only created if a synchronisation has been initiated.
* The size of the logs are user managed. 

Running the script produces considerable output informing the user of failures and successes of backups. The user can record this information by directing to a log file.
~~~
syncBKUP -s <source> -d <destination> > log_file
OR
syncBKUP -s <source> -d <destination> | tee log_file
~~~
### Backup MS Windows filesystem

Rsync was developed for use on *nix systems it was never intended for use on Windows file systems. Many advocate it is safe to use on Windows file systems but there are known issues with file permission and attributes when copying between Linux and Windows filesystems. I have seen rsync get the yips when synchronising data from WSL2-Linux to Windows attached USB storage. It pays to be vigilant and check for errors.

Running of syncBKUP on Microsoft WSL2 Linux and MSYS2 has different use cases. 
* **WSL2** is a virtual environment based on Hyper-V that has ready access to the Windows host. SyncBKUP can backup the data from the Linux side of the fence and the Windows side to Windows attached storage. 
* **MSYS2** is not a virtual environment it is based on the Unix-like Cygwin, it also provides easy access to Windows filesystem and attached devices.
* On Windows, WSL2 and MYSYS2 commands and scripts can be run from a Powershell terminal.    

Microsoft recommends that you do not run WSL2 on computers that run virtual hypervisors like VirtualBox, apparently there are conflicts between WSL2 and other virtual hypervisors. With computers that have VirtualBox or the like installed the safest option is to install MSYS2 to run syncBKUP.  

* **GitBash** is another Linux-like environment based on Cygwin, syncBKUP cannot be used on gitBash because rsync cannot be easily installed.
* **Cygwin**.  Cygwin is a more comprehensive environment than MSYS2, syncBKUP has not been tested in a Cygwin environment, it should work as advertised because the rsync installation package for MSYS2 came directly from Cygwin, it sill had the original name on the box.    

**RSYNC Options used**

* **-a** Archive mode
* **-A** Preserve ACLs
* **-X** Preserve extended attributes
* **-h** Human readable, for the logs
* **-v** Verbosity. Set to lowest verbosity used in individual logs.
* **--backup --backup-dir** These create the version directories
* **--no-links** Do not follow or use symbolic links.
* **--delete** At synchronisation, anything deleted on source is mirrored in the repository. Deleted files and directories saved in the version directories
* **--log-file** This appends the output created by rsync and the -v option to a designated log file.

Compression not used because it only of benefits IP network synchronisations. 
Rsync will not compress files that are already compressed (most multimedia) and small files.  

****Limitations**** - maximum Directory Path Length 

<ins>Linux</ins>

* Maximum characters permitted in a directory path obtained from the command **_getconf PATH_MAX /_** (usually 4096 characters).
* Changing the maximum Directory Path Length is not trivial, consult the internet for this one. 

<ins>MS Windows</ins>

* The default is 260 characters
* According Microsoft "The maximum path of 32,767 characters is approximate (sic)"
* See next section to determine current path length and change it.

**Setting Windows directory path length**

In Powershell (as administrator) run the _Get-ItemProperty_ command to determine if the maximum is enabled.  
~~~
PS C:\> Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled"

LongPathsEnabled : 0    <-----------------------  0 = not enabled otherwise its 1 = it is currently enabled 
PSPath           : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem
PSParentPath     : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control
PSChildName      : FileSystem
PSDrive          : HKLM
PSProvider       : Microsoft.PowerShell.Core\Registry
~~~
Use the _New-ItemProperty_ command to set it. Once done, restart the computer for the change to take effect.   
~~~
PS C:\> New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force

LongPathsEnabled : 1
PSPath           : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem
PSParentPath     : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control
PSChildName      : FileSystem
PSDrive          : HKLM
PSProvider       : Microsoft.PowerShell.Core\Registry
PS C:\>
~~~


     
     



   
   



   







