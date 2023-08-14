## Unraid ZFS Snapshot and Replication Script

**Overview**

This script is designed for Unraid servers version 6.12 and above, leveraging the ZFS capabilities introduced in this version. It provides a comprehensive solution for snapshotting and replicating ZFS datasets, either locally, to a remote server on the same network, or over the internet.

**Key Features:**

- **Automatic Snapshots:** The script can automatically snapshot the source_dataset based on a user-defined retention policy using Sanoid.
- **ZFS Replication:** The source_dataset can be seamlessly replicated to another ZFS zpool and/or dataset.
- **Rsync Replication:** If the destination isn't ZFS, the script can mount the snapshot and then use rsync for replication. Two modes are available:
  - **Mirror Copy:** A direct mirror of the source.
  - **Incremental Copy:** Uses hardlinks to create space-efficient incremental backups.
- **Remote Server Replication:** The script supports replication to a remote server, either on the same network or over the internet. For this functionality, SSH keys need to be exchanged between the source and destination servers to ensure secure and seamless data transfer.
- **Live Data Replication:** The script can replicate (backup) live data, such as Unraid docker appdata, without the need to stop the containers before backup.
- **Safety Checks:** The script is equipped with numerous safety checks to ensure smooth operation.
- **Notifications:** Users are informed of successes and errors through the Unraid notification system, which includes GUI and any other notifications the user may have set up.
- **Audible Tunes for Notifications:** If a beep speaker is present in the server, the script can provide optional audible notifications:
  - **Success in Snapshotting:** Plays the old Nokia ring tone.
  - **Success in Replication:** Plays the Mario achievement tune.
  - **Errors:** Plays the Star Wars Imperial march.

## Pre-requisites
Before using the script, ensure the following:

- Unraid server (version 6.12 or higher) with ZFS support.
- [User Scripts](https://forums.unraid.net/topic/48286-plugin-user-scripts/) plugin is installed.
- [Sanoid](https://forums.unraid.net/topic/94549-sanoidsyncoid-zfs-snapshots-and-replication/) plugin is installed.
- (Optional) [ZFS Master plugin](https://forums.unraid.net/topic/122261-plugin-zfs-master/) plugin is installed for enhanced ZFS functionality.
- Plugins are installed via Unraid's Community Apps

## Usage  -- Setting the Variables

To effectively utilize the script, you need to configure the main user-defined variables at the beginning of the script. These variables determine the behavior of the script, from snapshotting to replication. Below is a detailed breakdown of each variable:

### Notifications

- **`notification_type=`**: Determines when notifications are sent to the Unraid GUI and other notification systems.
  - **"all"**: Sends all notifications for both success and failure.
  - **"error"**: Sends notifications only for failures.
  - **"none"**: No notifications will be sent.
  
- **`notify_tune=`**: If set to "yes", an audible notification tune will be played on a beep speaker in the server (if available). Successful operations play the Mario "achievement tune", while failures play the Star Wars Imperial March tune. Set to "no" for no audible notifications. Set to "no" for no audible tune notifications.

### Source Configuration

- **`source_pool=`**: Name of the ZFS pool where your source dataset resides which you want to snapshot and or replicate. (Note this is not the mount point so DOESN'T start with /mnt).

- **`source_dataset=`**: Name of the dataset you intend to snapshot and/or replicate. If using auto snapshots, ensure the source_pool does not contain spaces due to Sanoid configuration limitations. If not using autosnapshots, source_dataset can contain spaces.

### Snapshot Settings

- **`autosnapshots=`**: Set to "yes" to enable automatic snapshotting of your source_dataset. Set to "no" for no snapshotting (if you only want replication)

- **Snapshot Retention Policy**: Determines how long snapshots are retained. These have default settings in the script and you can change them or leave as is.
  - **`snapshot_hours=`**: Number of hourly snapshots to retain.
  - **`snapshot_days=`**: Number of daily snapshots to retain.
  - **`snapshot_weeks=`**: Number of weekly snapshots to retain.
  - **`snapshot_months=`**: Number of monthly snapshots to retain.
  - **`snapshot_years=`**: Number of yearly snapshots to retain.

### Remote Server Configuration (For Remote Backups)

- **`destination_remote=`**: Set to "yes" if replicating to a remote server. Ensure the remote location is accessible and paired with SSH using shared keys. Set to "no" for local same server replications.

- **`remote_user=`**: Username for the remote server (for an Unraid server, this will typically be "root"). Not needed if location is local.

- **`remote_server=`**: Name or IP address of the remote server. Not needed if location is local.

### Replication Settings

- **`replication=`**: Determines the method of replication.
  - **"zfs"**: Use ZFS for replication using syncoid.
  - **"rsync"**: Use rsync for replication. This allows for replication to locations that are not ZFS.
  - **"none"**: No replication. You would set this to none if you only wanted the script to make snapshots.

### ZFS Replication Variables

- **`destination_pool=`**: Name of the ZFS pool for your destination (Note this is not the mount point so DOESN'T start with /mnt).

- **`parent_destination_dataset=`**: Name of the parent dataset for the replication to go to. A child dataset will be created within this (autonamed based on source_dataset).

- **`syncoid_mode=`**:  "strict-mirror" Strict mirroring that both mirrors the source and repairs mismatches (uses --force-delete flag).This will delete snapshots in the destination which are not in the source.
                         "basic" Basic replication without any additional flags will not delete snapshots in destination if not in the source

### Rsync Replication Variables

- **`parent_destination_folder=`**: Parent directory where a child directory (autonamed based off the source) will be created, containing the replicated data. (this does start with /mnt).

- **`rsync_type=`**: Determines the type of rsync replication.
  - **"incremental"**: Creates dated incremental backups.
  - **"mirror"**: Creates mirrored backups.

After setting these variables according to your requirements, you can run the script to perform snapshotting and replication as configured.



## Warning: Understanding and Safeguarding Your Data

Whether you're using the rsync function or ZFS replication in this script, it's important to understand how they work to ensure your data is safe. Here's a simplified explanation and precautions you can take to prevent accidental data loss:

- ### Simplified Working Principle:

Both the rsync function and ZFS replication in this script create separate, unique locations for your data to be transferred to. The location is based on your source pool and source dataset names, combing their names separated with an underscore. This means each source will have its own distinct location in the destination, reducing the chances of overwriting existing data.

For rsync, the location is created in the form `${parent_destination_folder}/${source_pool}_${source_dataset}`.

For ZFS replication, the location is `${destination_pool}/${parent_destination_dataset}/${source_pool}_${source_dataset}`.

- ### Precautions:

1. **Check your settings:** Always double-check your configurations before running the script. A small typo or error could lead to unintended consequences.
2. **Unique source names:** Make sure your source pool and dataset pairs have distinct names. This helps in creating unique locations at the destination and minimizes any conflicts.
3. **Manual modifications:** Avoid making changes manually in the destination datasets or folders after running the script. These could be overridden the next time you run the script.
4. **Maintain backups:** Always keep separate backups of your crucial data. Even though this script is designed to help you replicate and backup your data, having multiple backups gives you extra security.
5. **Enough free space:** Both rsync and ZFS need a certain amount of free space to perform operations. Ensure you have adequate free space in your destination locations.

Remember, while this script is designed with safeguards to reduce the chances of accidental data loss, it doesn't negate the need for caution and adherence to best data handling practices. Please makesure you understand what the script does and how it works before using it. And never underestimate the importance of a good backup!
