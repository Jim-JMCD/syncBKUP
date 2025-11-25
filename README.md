# UNDER CONTRUCTION 
## syncBKUP - Synchronised data backups with unlimited versioning. 

#### _SyncBKUP_ a Bash script to that synchronises data directories and files to a backup repository. Only the last backup is stored in the repository as a complete copy of the original.  All changes to a directory or file between backups are kept in individual version folders. The history of changes recorded version folders no limits.

### Usage
~~~
syncBKUP -s <source> -d <destination> -m <no-mount-point-check>
~~~
#### Options

**-s** Data source to be backed up can be either a single directory on the command line or a file listing directories to be backed up. 

**-d** The destination directory of the backup data, the backup repository.

**-m** Optional, when used it must be accompanied by "**no-mount-check**".   

**Input notes and restrictions**
* All directory paths must be <ins>full paths</ins>.
* <ins>All symbolic links</ins> are ignored and will not be processed.
* Single directory input with spaces require single quotes **'/mnt/c/Users/ted/My Documents'**
* Trailing slahes on directory names are ignored, eaxmple: **/home/ada/** will be preocessed as **/home/ada**    

**Source File Format**

A source file provides a list of directtories to back up. The format requirments are 
* Any line that starts with *'#'* is ignored.
* Any line that starts with *'.'* (fullstop) terminates backup processing. Any directories listed after that line will not be processd.
* Blanks lines are permitted.
* Leading spaces on line entries permitted.
* Only lines that only contain a single valid non-empty directory will be processed, <ins>any additional material on a line will cause the line not to be processed</ins>. 
   
**Mount Point Check**
  
A safety check to prevent the potential of the root and other fiesystems filling up. It also pays to check using **df -Th** command

Traditionally */mnt* is where temporary external devices and network shares are attached, with each device having its own directory.  To attach (aka __mount__) a device a pre-existing directory (aka __mount point__) must exist it usually has flexible write permissons. Any utlitiy that copies data to the mount point without a device attached will run as normal and <ins>could fill up the root filesystem</ins> or what ever filesystem the mount point is on. When the mount point is used by an external device, the directories and files that were previosly written to it remain hidden and using up space. 

* The mount point check covers the first three directories of a backup destination path, example **/mnt/USB/BKUP_2TB/syncBKUP/2025** the directories **/mnt**, **/mnt/USB** and **/mnt/USB/BKUP_2TB** will checked. If none of those directories contian a mounted device the check will fail and processing will not proceed.
* Symbolic links to mount points are often used to create a convenient refence to storage devices. If the symbolic link is straight forward, it will pass the mount point check if a storage device is attached to the mount point that it references. With symbolic links there are no guaratees, it pays to check.
* Special case: In a MSYS2 environment the mount point check is disabled.
   
_Disabling Mount Point Check_ : The option -m must be accompanied with _no-mount-check_ : **-m no_mount_check**

### Backup Destination 

Directory strucuture of the backp repository, the levels are:
* Level 1 - Top level - an existing directory given to syncBKUP as **-d \<destination\>**
* Level 2 - The computer name where the script was run. This permits multiple computers tp write to the same storage area.
* Level 3 - The directory that contains and uniquely each directory that is backed up i.e. given to syncBKUP as **-s <source>**. This directory contains the version directories and the log for each backup.  
* Level 4 - The directory level named after the parent of the data. Example this directory will be named **music** when given **-s /home/ada/music** to backup. All directories below this level contain the backup data.

#### Identifying what is backed up 
Level 3, the directory name uniquely identifies each directory backed up by created a direcotry named using the directory path of of the source.  The "/" slashes and spaces of the source directory are replaced with underscores "_".

All backups can be located in **../Computer name/Modified_source_directory_name/..* 
~~~
Computer name = star03
Data source   = -s /home/ada/music/2010s/dubstep
Destination   = -d /mnt/USB/BKUP_2TB/syncBKUP/2025
Location /mnt/USB/BKUP_2TB/syncBKUP/2025/star03/home_ada_music_2010s_dubstep/dubstep/
~~~
* The synchronised data backup repository is
  * **.../star03/home_ada_music_2010s_dubstep/dubstep/**
* Logs for /home/ada/music/2010s/dubstep backup are located in the same directory as **.../dubstep/**
* Version directories are located in the same as the logs and **.../dubstep/**
~~~
Computer name = fred02
Data source   = -s '/mnt/c/Users/ted/Music/1990 to 1997/Shoegaze and Nu Metal'
Destination   = -d /mnt/USB/BKUP_2TB/syncBKUP/2025
Location '/mnt/USB/BKUP_2TB/syncBKUP/2025/fred02/mnt_c_Users_ted_Music_1900_to_1997_Shoegaze_and_Nu_Metal/Shoegaze and Nu Metal'
~~~
* The synchronised data backup repository is
  * **.../fred02/mnt_c_Users_ted_My_Music_1900_to_1997_Shoegaze_and_Nu_Metal/Shoegaze and Nu Metal/**

* Logs for /home/ada/music/2010s/dubstep backup are located in the same directory as **.../Shoegaze and Nu Metal/**
* Version directories are located in the same as the logs and **.../Shoegaze and Nu Metal/**
* Note: Directories names with spaces have the spaces converted to underscores. 
~~~
'/ted/Music/1990 to 1997/Shoegaze and Nu Metal'
Is converted to
ted_Music_1990_to_1997_Shoegaze_and_Nu_Metal
~~~
Maximum characters permitted in a directory path obtained from the command _getconf PATH_MAX /_ usually 4096 charcaters.

### LOGs

   
   



   







