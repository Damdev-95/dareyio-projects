## LINUX COMMANDS IN DAREY.IO LESSON
* `sudo` superuser do 
* `cd` change directory
* `mkdir` create directory
*  `rmdir` delete directory
*  `cat` to display contents of a file
*  `mkdir -p folder_1/folder_2/folder_3` create a nested folder 
* `ls -la` long listing of all files
* `touch` create a file
* `pwd` present work directory
* `which` where is the command located
* `rm` delete file
* `rm -r` delete recursively (All files in folder)
* `man` manual
* `echo` print out
* `lsof -i :3000` list open files with port 3000
* `kill -9 PID` delete the process with the PID
* `mkdir folder1 && vim folder1` execute both commands together
* `sudo chown -R $USER:$USER /folder_path` To change ownwership of folder 
* `tree folder_name` to check the file structure of the folder 
* `du` to check the size of file 
* `tree \ -L 1` to check the file structure of root directory with level 1
* `bin` directory that contains binary files of the program codes
* `boot` directory that has information relating to the boot of the OS
* `dev` directory on device files, which is an interface to a device driver
* `etc` directory where configuration files are kept for application.
* `home` directory where users keep specific files and folders *user's domain*
* `lib` directory for collection of resources used by system, *.so* extension for modules
* `lost+found` directory for system recovery
* `mnt` for mounting external drives or disks.
* `opt` option directory for custom softwares on the Linux.
* `proc` process is an instance of a program, shows the proces `ls -l /proc | grep PID`
* `root` directory for the root user
* `run` temporary data used by process are kept there
* `df -h` *disk free* to find information about space on disk
* `free -h` to find information about the system memory
* `id` Id for the user on the Linux system `ls /run/user`
* `sbin` system binaries
* `snap` package manger related to snap(Linux OS), similar to apt
* `srv` service files are kept .
* `sys` mounted on a virtual file system, 
* `tempfs` file storage on the system memory
* `tmp` temporary files and directories are kept on disk
* `usr` program files are kept in this directory
* `var` directory that stores data written on the system such as log ,spool(email, printer)
* `diff` to display the difference between two files 
* `sudo useradd -m <username>` to create a user with home directory 
* `sudo passwd <username>` to create a password for the user 
* `sudo grep username /etc/passwd` or `cat /etc/passwd | grep username`
