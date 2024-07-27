English | [Deutsch](autofs.md)

# SMB shares with Autofs
Autofs can be used to automatically mount remote server shares as required and, if necessary, automatically unmount them again after a previously defined time. 

## Installation of Autofs

`sudo apt-get install autofs`

## .smbcredentials

If a password is required for sharing, a hidden text file with the name .smbcredentials must be created in your own user home folder.

- Create a new, empty file with the name .smbcredentials  
    `touch ~/.smbcredentials`  

- Edit the .smbcredentials file just created    
    `nano ~/.smbcredentials`  

- Contents of the file ~/.smbcredentials  
    ```
    username=[USERNAME]
    password=[PASSWORD]
    ```    
- Restrict the file permissions of the ~/.smbcredentials file  
    `sudo chmod 600 ~/.smbcredentials`
    
## The master map file

- Specify the mount point where the shares should be mounted  
    `sudo mkdir -p /media/[USERNAME]/[MOUNTPOINT]`  

- Enter the following line at the end of the /etc/auto.master file  
    ***Syntax:** \[mountpoint\] \[map-file\] \[options\]*  
    `/media/[USERNAME]/[MOUNTPOINT] /etc/auto.[FILENAME] --timeout=0 --ghost`

## The map file

- Create a new, empty file  
    `sudo touch /etc/auto.[FILENAME]`  

    ...or more simply...  
    `sudo nano /etc/auto.[FILENAME]`  

- Remove the executability of a file /etc/auto.\[FILENAME\]  
    `sudo chmod -x /etc/auto.[FILENAME]`

     **Attention:** Files with this structure must not be marked as executable.

- Contents of the file /etc/auto.\[FILENAME\]  
    ***Syntax:** \[SHARENAME\] -fstype=cifs,\[OPTIONS\] ://\[IP-ADDRESS\]/\[SHAREPATH\]*  
    ```
    [SHARENAME] -fstype=cifs,uid=1000,credentials=/home/[USERNAME]/.smbcredentials ://[IP-ADDRESS]/[SHAREPATH]
    [SHARENAME] -fstype=cifs,uid=1000,credentials=/home/[USERNAME]/.smbcredentials ://[IP-ADDRESS]/[SHAREPATH]
    ```

You can find more information about autofs at https://wiki.ubuntuusers.de/Autofs/
