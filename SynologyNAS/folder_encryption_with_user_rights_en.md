English | [Deutsch](https://github.com/toafez/Tutorials/blob/main/folder_encryption_with_user_rights.md)

## Mount and unmount encrypted folders as DSM user
With the monitoring script presented here, encrypted folders on your Synology NAS can also be mounted and unmounted by users who do not belong to the administrators group.

## How the monitoring script works
After the **one-time execution** of the **monitoring script** via the **DSM task scheduler** as user "**root**", it is first checked **in the root directory** of a previously specified **user home folder** whether a so-called "**event file**" already exists. If no "event file" is found, it is created by the script with a freely definable file name. The "event file" is then **monitored** by the monitoring script for **saved content changes**.

Freely definable "**trigger values**" for mounting and unmounting encrypted folders and for ending the monitoring are **written** into the "event file". If a valid trigger value is recognized by the monitoring script after saving, a corresponding action is performed.

**Important note:** Editing the "event file" is only possible via the DSM text editor.

## Installation instructions
- ### Activate user home service   
    Please activate the **User Home service** in advance. To activate the user home service, go to **DSM main menu** > **Control Panel** > **User and Group** and switch to the tab > **Advanced**. Activate the **Enable user home service** checkbox under the **User base** menu item.

- ### Install DSM text editor
    Install the **Text Editor** via the **DSM Package Center** to be able to edit the "event file" within the DSM later on. Make sure that the desired DSM user is also allowed to use the text editor. You can set this accordingly via **Main menu > Control panel > Application permissions**.

- ### Install inotify-tools
    Also install the [inotify-tools](https://synocommunity.com/package/inotify-tools) package from the [SynoCommunity](https://synocommunity.com/) via the **DSM package center**.

- ### Add the monitoring script to the task scheduler
    - Log in to the DSM of your Synology NAS as **Adminstrator**.
    - Open the Task Scheduler in DSM under **Main Menu > Control Panel**.
    - Select the **Create > Scheduled task > Custom script** button in the task scheduler.
    - In the pop-up window that now opens, give the task an individual name in the **General > General settings** tab and select **root** as the user.
    - Then add the **monitoring script** to the text field in the **Task settings > Execute command > Custom script** tab...
    - Confirm your entries with the **OK** button and accept the subsequent warning message with **OK**.
    - To add the task to the task planner, you must then enter the password of the user currently logged in to DSM and confirm the process by clicking the Send button.
    - In the task planner overview, you can now **mark** the task you have just created with the mouse (the line should be highlighted in blue after marking) and trigger the task manually once by pressing the **Execute** button.

## The monitoring script
Before you add the monitoring script to the task planner, replace the placeholders written in capital letters...

- **USERNAME** ... with the name of the desired DSM user e.g. **tommes**
- **FOLDERNAME**... with the folder name of the encrypted folder e.g. **secret**
- **PASSWORD**... with the password required to mount the encrypted folder
- **CONNECT**... with a trigger value for mounting the encrypted folder, e.g. **1**, **mount** or **connect**
- **DISCONNECT**... with a trigger value to unmount the encrypted folder e.g. **0**, **unmount** or **disconnect**
- **EXIT**... with a trigger value to end the monitoring as well as the monitoring script itself, e.g. **100** or **exit**.
- **EVENTFILE**... with a freely definable file name.

    **Note:** Please do not use umlauts, special characters or spaces!!!

```
#!/bin/bash

# DSM user name (represented in the user home folder)
user="USERNAME"

# Name of the encrypted, shared folder
share="FOLDERNAME"

# Encryption key
password="PASSWORD"

# Trigger value to mount an encrypted folder
decrypt_trigger="CONNECT"

# Trigger value to disconnect an encrypted folder
encrypt_trigger="DISCONNECT"

# Trigger value to end the monitoring
exit_trigger="EXIT"

# Path and file name of the event file
eventpath=$(echo /volume[[:digit:]]/homes/${user})
eventfile=$(echo ${eventpath}/EVENTFILE.txt)

if [ -d "${eventpath}" ]; then
	if [ ! -f "${eventfile}" ]; then
	    touch "${eventfile}"
        chown "${user}":"users" "${eventfile}"
	    echo "value" > "${eventfile}"
	fi
else
	exit 1
fi

if [ -f "${eventfile}" ]; then
    echo "Überwachung läuft..."
    inotifywait -mq --format %w%f ${eventfile} -e close_write | while read event
    do
        if [ $(cat "${event}") == "${decrypt_trigger}" ]; then
            if [ -n "/volume[[:digit:]]/@${share}@" ] && [ -n "${password}" ]; then
                /usr/syno/sbin/synoshare --enc_mount ${share} ${password}
                echo "Der verschlüsselte Ordner [ ${share} ] wurde eingehängt."
            fi
        elif [ $(cat "${event}") == "${encrypt_trigger}" ]; then
            if [ -n "/volume[[:digit:]]/${share}" ]; then
                /usr/syno/sbin/synoshare --enc_unmount ${share}
                echo "Der verschlüsselte Ordner [ ${share} ] wurde getrennt."
            fi
        elif [ $(cat "${event}") == "${exit_trigger}" ]; then
            pid=$(ps aux | grep -v "grep" | grep -E "inotifywait.*--format.*${eventfile}.*close_write" | awk -F' ' '{print $2}')
            kill ${pid}
        	exit 0     
        fi
    done
else
	exit 2
fi
```
