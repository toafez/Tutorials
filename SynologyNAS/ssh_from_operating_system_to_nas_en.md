English | [Deutsch](ssh_from_operating_system_to_nas.md)

### This tutorial contains the supplementary video description of my YouTube video... ###
# **[Set up SSH public key connection to a Synology NAS](https://youtu.be/VjoWjX_8E3Q)** #

## **On your Windows, Linux, Mac or Whatever operating system**
First of all, we need a terminal program to access the console of our Windows, Linux, Mac or whatever operating system. Once on the console, we want to briefly orient ourselves...
- **Who am I**

    ```whoami```

- **Where am I actually located**

    `pwd`

    _(Print Working Directory)_

- **Show folder contents**

    `ls -la`

    _(Option **l** = file information is displayed in long form)_

    _(Option **a** = also shows hidden files and folders beginning with a dot)_

- Create a new, hidden folder with the name .ssh and assign folder rights**

    `mkdir -p ~/.ssh`
  
    `chmod 0700 ~/.ssh`

- **The meaning of tilde (~)**

    The **tilde** symbol (**~**) in the Linux file system represents the current user's home directory. It is a shorthand notation that allows users to point to their own home directory without having to enter the full path.

- Create **RSA key pair**

    By simply entering the command **ssh-keygen**, different encryption algorithms can be used depending on the configuration. Therefore, always specify which algorithm you want to use.

    `ssh-keygen -b 4096 -t rsa -f ~/.ssh/id_rsa`

    _(**-b** stands for bit, i.e. the bit length of the encryption key, here 4096 bits)_

    _(**-t** stands for type, i.e. the encryption type, here rsa for RSA protocol 2)_

    _(**-f** stands for the folder path and file name to be used. The default file name id_rsa is used here as an example)_

- Output the content of the public key**

    `cat ~/.ssh/id_rsa.pub`

- **Create a file with the name authorized_keys and add your own public key**

    `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

    _(**cat** = concatenate)_

    _(command **>** file = (overwrite) standard output of the command in the target file)_

    _(Command **>>** file = append standard output of the command to the target file.)_

- Set folder and file permissions**

    With **chmod** you can change the access rights of folders and files.

    `chmod 0700 ~/.ssh`

    `chmod 0600 ~/.ssh/id_rsa*`

    `chmod 0600 ~/.ssh/authorized_keys`

- **Establish SSH connection to your Synology NAS**

    **Note:** Please activate the SSH terminal service and the user home service on your Synology NAS in advance.

    **Syntax:** ssh -p [PORT] [USER]@[NAS-IP]

    **Example:** `ssh -p 22 tommes@172.16.1.11`

    _(Accept connection by handshake and enter user password)_

## On your Synology NAS

- **Where am I located anyway**

    `pwd`

    _(Print Working Directory)_

    _(You are in the user home folder of your administrator under /var/services/homes/[USER] or /volume1/homes/[USER)_

- Create a new, hidden folder with the name .ssh and assign folder rights**.
    `mkdir -p ~/.ssh`

    `chmod 0700 ~/.ssh`

    `exit`

## Back on your Windows, Linux, Mac or Whatever computer

- Append the contents of the id_rsa.pub file to the authorized_keys file via an SSH connection to your Synology NAS**

    `cat ~/.ssh/id_rsa.pub | ssh -p [PORT] [USER]@[NAS-IP] “cat >> ~/.ssh/authorized_keys”`

- **New connection to our Synolog NAS, but this time without entering a password**

    `ssh -p [PORT] [USER]@[NAS-IP]`

    _(login without password)_

## Again on your Synology NAS

- **Set file permissions**

    `chmod 0600 ~/.ssh/authorized_keys`

- **Switch to root account**

    `sudo -i`

    _(Enter the administrator's password repeatedly and perform a handshake)_

- Where am I anyway**

    `pwd`

    _(Print Working Directory)_

    _(You are in root's home folder under /root)_

- **Create a new, hidden folder with the name .ssh and assign folder rights**
    `mkdir -p ~/.ssh`
  
    `chmod 0700 ~/.ssh`

- **Copy the contents of the id_rsa.pub file from your administrator's user home folder to the root folder**

    `cp /var/services/homes/[USER]/.ssh/id_rsa.pub /root/.ssh/authorized_keys`

    ... or alternatively ...

    `cp /volume1/homes/[USER]/.ssh/id_rsa.pub /root/.ssh/authorized_keys`

- **Set file permissions**
  
    `chmod 0600 ~/.ssh/authorized_keys`
