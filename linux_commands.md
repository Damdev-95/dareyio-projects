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
* `inode` stores the metadata about a file 
* `file table` stores the file name and inode number
* `df -i` this indicates the size of inodes for a specific disk or partition 
* `ls -i filename` to indicate the inode number of the file or directory
* `stat filename` for details on a file
* `cat authorize_keys > id_rsa.pub` to overwrite the contents of public key with the given authorized keys
* `sudo useradd -m username -G groupname` add new users in a group
* `sudo groupadd groupname` to create a new group 
* `sudo passwd username` to enter a password for username
* `sudo usermod -a -G groupname username` modify username with a group
* `sudo userdel`
* `tar -cvf archived.tar file` to aarchive file
* `tar -xvf archived.tar` to extract from an archive
* `tar -czvf archived.tar.gz file` to archive and zip file
* 
```
bzip2 archived.tar
bunzip2 archived.tar.bz2
zip archived.tar.zip archived.tar
zip -p password@123 folder.zip folder
unzip archived.tar.zip
```
* `du -h` to view the disk usage in human readable format
* `snap` comes in due to the framented nature of linux distros, debian and redhat
* using snap tool to install software irrespective of the distros
* `yum` interact with the redhat package
* payload is the actual data within a packet
* SSH - secure shell, being able to connect to remote server in a secure way
* A key pair needs to exist, public key and private key 
* * `ssh-keygen -t rsa` generate ssh keys 
* `ssh-copy-id -i /home/ubuntu/.ssh/id_rsa.pub  remote_username@public-ip` to transfer ssh public key to the remote server from the client.
