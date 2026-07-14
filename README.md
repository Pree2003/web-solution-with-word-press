<img width="1096" height="352" alt="image" src="https://github.com/user-attachments/assets/e56b2020-eb26-4a67-9312-c1d875a9a3ae" />

After creating the three 10 GiB Amazon Elastic Block Store (EBS) volumes in the same Availability Zone as the Web Server EC2 instance, each volume was attached individually to the Red Hat Enterprise Linux Web Server using the AWS Management Console.
To ensure successful attachment, each EBS volume was assigned a unique device name during the attachment process as follows:
EBS VOLUME	DEVICE NAME
Volume 1	/dev/sdf
Volume 2	/dev/sdg
Volume 3	dev/sdh

<img width="542" height="257" alt="image" src="https://github.com/user-attachments/assets/54081c73-e501-4eb8-aae0-a5125863357f" />

The output verifies that:
The root disk (nvme0n1) contains the Red Hat Enterprise Linux operating system.
Three additional 10 GiB EBS volumes (nvme1n1, nvme2n1, and nvme3n1) were successfully attached to the Web Server instance.
The additional disks are available for the next stage of the project, which involves partitioning the disks and configuring storage.

<img width="579" height="190" alt="image" src="https://github.com/user-attachments/assets/2144ac86-5977-438c-a87c-ca5eedc1f294" />

After confirming that the operating system detected the attached EBS volumes using the lsblk command, the df -h command was executed to display the mounted file systems and their disk usage.
The output confirms that the Red Hat Enterprise Linux operating system is installed on the root partition (/dev/nvme0n1p3), which is mounted on the root directory (/). At this stage, only the operating system partitions are mounted because the three newly attached EBS volumes have not yet been partitioned, formatted, or mounted. These additional volumes will be configured in the subsequent steps of the project for RAID and WordPress data storage. 

<img width="554" height="442" alt="image" src="https://github.com/user-attachments/assets/609b3bbd-1e2e-4109-a75f-68f3886958f3" />

After verifying that the three EBS volumes were successfully attached to the Web Server instance, the next step was to partition the disks. The project tutorial used the gdisk utility; however, on the Red Hat Enterprise Linux 10.2 instance used for this project, the gdisk package was not available. As an alternative, the parted utility was used to create a GUID Partition Table (GPT) and a single primary partition on each disk.
The first EBS volume (/dev/nvme1n1) was partitioned using this commands:
sudo parted /dev/nvme1n1 

Within the parted interactive utility, the following commands were executed:
mklabel gpt
mkpart primary 0% 100%
print
quit
The print command confirmed that a GPT partition table had been created successfully and that a single primary partition occupied the entire disk.

<img width="501" height="350" alt="image" src="https://github.com/user-attachments/assets/d37ca60d-644b-40a3-b95f-03c971b6d1d8" /> 

After successfully partitioning the first EBS volume, the same procedure was repeated for the second EBS volume (/dev/nvme2n1) using the GNU Parted utility. A GPT partition table was created, followed by a single primary partition occupying the entire disk.
The following command was executed:sudo parted /dev/nvme2n1

<img width="449" height="345" alt="image" src="https://github.com/user-attachments/assets/69534b64-c91d-463d-bfec-d425706050a6" />

The partitioning procedure was then repeated for the third EBS volume (/dev/nvme3n1). A GPT partition table and one primary partition were created using the GNU Parted utility.
The following command was executed:sudo parted /dev/nvme3n1

<img width="373" height="231" alt="image" src="https://github.com/user-attachments/assets/290a66ac-716c-45b1-b30d-2259780ea910" />

After partitioning all three EBS volumes, the lsblk command was executed to verify that the Linux operating system recognized the newly created partitions.

<img width="553" height="141" alt="image" src="https://github.com/user-attachments/assets/b6c13aec-7b79-4f43-a339-1278713382d4" />

The Logical Volume Manager (LVM2) package was verified on the Red Hat Enterprise Linux Web Server to ensure that the tools required for creating and managing logical volumes were available. The yum package manager was used to install the package. The output indicated that LVM2 was already installed on the system; therefore, no additional installation was required. This confirmed that the server was ready for the subsequent stages of configuring physical volumes, volume groups, and logical volumes.
The following command was executed:
sudo yum install lvm2

<img width="342" height="185" alt="image" src="https://github.com/user-attachments/assets/c6c6ff28-b743-49c1-afb1-9c21593bc206" />

After confirming that the LVM2 package was available, the lvmdiskscan utility was executed to scan the system for disks and partitions that could be used for Logical Volume Manager (LVM) configuration. The scan identified the root system partitions as well as the three newly created partitions on the attached EBS volumes (/dev/nvme1n1p1, /dev/nvme2n1p1, and /dev/nvme3n1p1). At this stage, the output indicated that no LVM physical volumes had been created, which was expected because the partitions had not yet been initialized as physical volumes. This verification confirmed that the storage devices were correctly partitioned and ready for the next step of creating LVM physical volumes.
The following command was executed:
sudo lvmdiskscan

<img width="428" height="123" alt="image" src="https://github.com/user-attachments/assets/505c7396-1c0f-4b42-9749-b3c8308b33a0" />

After partitioning the three attached EBS volumes, each partition was initialized as a Logical Volume Manager (LVM) Physical Volume (PV) using the pvcreate command. This step prepares the partitions for inclusion in a Volume Group (VG), which is the next stage of the LVM storage configuration.
The following commands were executed:
sudo pvcreate /dev/nvme1n1p1
sudo pvcreate /dev/nvme2n1p1
sudo pvcreate /dev/nvme3n1p1

<img width="330" height="111" alt="image" src="https://github.com/user-attachments/assets/a1266b8e-a1f8-4285-a69e-82b9c5069331" />

After creating the physical volumes, the pvs command was executed to verify that LVM successfully recognized them. The output confirmed that the three partitions had been initialized as physical volumes and that each contained approximately 10 GiB of unallocated space available for use in a Volume Group.
The following command was executed:sudo pvs

<img width="522" height="240" alt="image" src="https://github.com/user-attachments/assets/4019eb63-ce55-453b-99b2-43d1f21e63ef" />

After successfully creating the three LVM physical volumes, the next step was to create a Volume Group (VG) named webdata-vg. During the initial attempt, the Volume Group was created using only the first physical volume (/dev/nvme1n1p1). As a result, subsequent attempts to execute the vgcreate command returned a message indicating that the Volume Group already existed.

The following command was executed:

sudo vgcreate webdata-vg /dev/nvme1n1p1

The system returned the following confirmation:

Volume group "webdata-vg" successfully created

Since the Volume Group had already been created, the remaining physical volumes were added using the vgextend command instead of vgcreate.

The following commands were executed:

sudo vgextend webdata-vg /dev/nvme2n1p1
sudo vgextend webdata-vg /dev/nvme3n1p1

The system confirmed that the Volume Group was successfully extended to include all three physical volumes.

<img width="351" height="76" alt="image" src="https://github.com/user-attachments/assets/63887a0b-5793-4bce-9bff-cdb2e38df741" />

After creating and configuring the Volume Group, the vgs command was executed to verify that the Logical Volume Manager (LVM) successfully recognized the Volume Group and its associated physical volumes. The verification confirmed that the Volume Group webdata-vg was created successfully and consisted of three physical volumes, with all storage space available for future logical volume allocation.
The following command was executed:
sudo vgs
<img width="638" height="158" alt="image" src="https://github.com/user-attachments/assets/e16029eb-f4be-46a3-aa2f-c20ae1c16a6b" />

After successfully creating the Volume Group (webdata-vg), the next step was to create Logical Volumes (LVs). Two logical volumes were created from the available storage pool within the Volume Group. The first logical volume, named apps-lv, was allocated 14 GiB of storage to host the web application data. The second logical volume, named logs-lv, was also allocated 14 GiB of storage and was designated for storing log files. Together, these logical volumes provide a structured and flexible storage layout that separates application data from system logs.
The following commands were executed:
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg

The system successfully created both logical volumes, confirming that the requested storage was allocated from the webdata-vg Volume Group.

<img width="639" height="207" alt="image" src="https://github.com/user-attachments/assets/3104384b-f61b-4253-ac7c-5c394cb57a43" />

After creating the Physical Volumes (PVs), Volume Group (VG), and Logical Volumes (LVs), the complete Logical Volume Manager (LVM) configuration was verified. Verification was performed using the pvs, vgs, and lvs commands to ensure that all storage components were correctly configured and linked together.
The following commands were executed:
sudo pvssudo vgssudo lvs
The verification confirmed that the three physical volumes were successfully added to the webdata-vg Volume Group and that two logical volumes, apps-lv and logs-lv, had been created with a capacity of 14 GiB each. The Volume Group provided approximately 30 GiB of storage, with the remaining unallocated space reserved for future expansion.

<img width="444" height="294" alt="image" src="https://github.com/user-attachments/assets/4a518799-90b9-4aa3-b741-0ea55335c828" />

After verifying the Logical Volume Manager (LVM) configuration, the lsblk command was executed to display the complete storage hierarchy of the system. This command provides a tree view of the block devices, including physical disks, partitions, and logical volumes.
The following command was executed:
sudo lsblk
The output confirmed that the three attached EBS volumes had been partitioned and incorporated into the webdata-vg Volume Group. It also showed the logical volumes apps-lv and logs-lv, verifying that the storage hierarchy had been configured successfully and was ready for filesystem creation and mounting.

<img width="558" height="435" alt="image" src="https://github.com/user-attachments/assets/7253d5c2-642c-4897-a051-7029418bb67a" />

After creating the logical volumes, the next step was to create a filesystem on each volume so that they could be mounted and used by the operating system. The ext4 filesystem was selected because it is the default and one of the most widely used journaling filesystems on Linux, offering reliability, stability, and good performance.

The logical volume apps-lv was formatted first using the following command:

sudo mkfs.ext4 /dev/webdata-vg/apps-lv

The command successfully created an ext4 filesystem and generated a unique filesystem UUID for the logical volume. During the formatting process, the filesystem metadata, inode tables, journal, and superblocks were created successfully, indicating that the logical volume was ready for use.

The second logical volume, logs-lv, was then formatted using the following command:

sudo mkfs.ext4 /dev/webdata-vg/logs-lv 

This command also completed successfully, creating an ext4 filesystem on the logical volume and assigning it a unique filesystem UUID. The successful completion of both commands confirmed that the logical volumes had been prepared for mounting.

Formatting the logical volumes is a critical step because it transforms the raw storage provided by the Logical Volume Manager into filesystems that Linux can mount and use for storing application data and log files.

<img width="554" height="464" alt="image" src="https://github.com/user-attachments/assets/0e4f9677-bd0c-454d-a3dd-e578a09f2817" />

Before mounting the logs-lv logical volume, the contents of the system log directory (/var/log) were backed up to the /home/recovery/logs directory using the rsync utility. This precaution ensures that the existing log files are preserved because mounting a filesystem on a directory temporarily hides any files already stored in that directory.
The following command was executed:
sudo rsync -av /var/log/ /home/recovery/logs/
The rsync command successfully copied the contents of the /var/log directory, including system logs, audit logs, package management logs, authentication logs, and other important log files. The -a (archive) option preserved file permissions, ownership, timestamps, symbolic links, and directory structure, while the -v (verbose) option displayed detailed information about the synchronization process.
A second execution of the rsync command synchronized only the log files that had changed since the initial backup, demonstrating rsync's incremental synchronization capability and ensuring that the backup contained the most recent log data before the logical volume was mounted.

<img width="553" height="215" alt="image" src="https://github.com/user-attachments/assets/2a6553e3-2ed4-4a69-be60-d3803344d425" />

<img width="554" height="35" alt="image" src="https://github.com/user-attachments/assets/84a7cef8-34db-419e-8b21-20827570454f" />

After backing up the log files, the logs-lv logical volume was mounted to the /home/recovery/logs directory. Mounting the logical volume associates the newly created ext4 filesystem with the designated mount point, providing dedicated storage for log data.
The following command was executed:
sudo mount /dev/webdata-vg/logs-lv /home/recovery/logs/
The command completed successfully without any errors, indicating that the logs-lv logical volume had been mounted successfully. From this point onward, the /home/recovery/logs directory uses the storage allocated to the logs-lv logical volume.
Mounting a dedicated logical volume for log storage helps separate application data from system logs, making storage management more organized and improving maintainability and data recovery.

<img width="554" height="511" alt="image" src="https://github.com/user-attachments/assets/a82d59e5-2c7b-4f0d-8e6a-80e5a35fc807" />

After backing up the existing log files, the logs-lv logical volume was prepared for its permanent mount location. The logical volume was initially unmounted from the temporary backup directory (/home/recovery/logs) and then mounted to the /var/log directory, which is the default location for Linux system and application log files.

The following commands were executed:

sudo umount /home/recovery/logs
sudo mount /dev/webdata-vg/logs-lv /var/log

The commands completed successfully without any errors, confirming that the logs-lv logical volume had been mounted to /var/log. Mounting the logical volume to this directory provides dedicated storage for log files, improving storage organization and allowing the log data to be managed independently from the operating system and web application files.

The following command was executed:

sudo mount /dev/webdata-vg/logs-lv /var/log

<img width="472" height="63" alt="image" src="https://github.com/user-attachments/assets/2f1ff8a1-68b1-4d14-acc9-b46b15c483a8" />


Before mounting, the logical volume was unmounted from its temporary backup location (/home/recovery/logs) to allow it to be mounted at its intended destination. Once mounted, the /var/log directory began using the dedicated storage provided by the logs-lv logical volume.

Because mounting a filesystem hides the previous contents of the mount point, the earlier backup of the /var/log directory was essential. The backed-up log files can now be restored to the newly mounted logical volume, ensuring that existing system logs remain available while benefiting from dedicated storage.

<img width="553" height="145" alt="image" src="https://github.com/user-attachments/assets/108ba81d-831d-4019-a403-592fda7c04ad" />


To ensure that the logical volumes would be mounted automatically after every system reboot, the UUIDs (Universally Unique Identifiers) of the newly created ext4 filesystems were retrieved using the blkid command. UUIDs provide a persistent method of identifying storage devices, making the mounting process more reliable than referencing device names, which may change after a system restart.
The following command was executed:
sudo blkid
The output displayed information about all block devices attached to the instance, including their UUIDs and filesystem types. The UUIDs of the apps-lv and logs-lv logical volumes were identified and recorded for use in the /etc/fstab configuration file.
Using UUIDs instead of device names ensures that the correct logical volumes are mounted consistently, even if Linux assigns different device names after a reboot

<img width="554" height="317" alt="image" src="https://github.com/user-attachments/assets/58a0edf8-af74-498e-ac29-83589ca13b65" />

To ensure that the logical volumes are mounted automatically whenever the server restarts, the /etc/fstab configuration file was updated using the UUIDs of the newly created ext4 filesystems. UUIDs provide a persistent method of identifying storage devices and are preferred over device names because device names may change after a system reboot.
The /etc/fstab file was opened using the following command:
sudo vi /etc/fstab
The UUIDs of the apps-lv and logs-lv logical volumes were then added to the file, associating each filesystem with its respective mount point (/var/www/html and /var/log). After saving the file, the system configuration was reloaded to recognize the updated mount settings.
The following commands were executed:
sudo systemctl daemon-reloadsudo mount -a
The mount -a command completed successfully without any errors, confirming that the /etc/fstab configuration was valid and that the logical volumes could be mounted automatically during system startup.
To verify the configuration, the df -h command was executed:
df -h
The output confirmed that both logical volumes had been mounted successfully. The apps-lv logical volume was mounted on /var/www/html, while the logs-lv logical volume was mounted on /var/log. This verification demonstrated that the storage configuration was functioning correctly and would persist after future server reboots.

<img width="554" height="131" alt="image" src="https://github.com/user-attachments/assets/265e1f17-e98e-4e47-a040-8df4ece9cbb7" />

After creating the three 10 GiB Amazon Elastic Block Store (EBS) volumes in the same Availability Zone as the Web Server EC2 instance, each volume was attached individually to the Red Hat Enterprise Linux Web Server using the AWS Management Console.To ensure successful attachment, each EBS volume was assigned a unique device name during the attachment process. 

<img width="554" height="144" alt="image" src="https://github.com/user-attachments/assets/ac5d2386-cf26-44d4-82c0-541905d61a27" />

<img width="553" height="129" alt="image" src="https://github.com/user-attachments/assets/2fdd9673-9863-4fd7-b9fb-d4943e8cbc20" />

To allow the WordPress web server to communicate securely with the remote MariaDB database, an inbound security group rule was configured on the DB Server. The MySQL/Aurora service was selected, which automatically uses TCP port 3306, the default port for MySQL and MariaDB database connections. For security purposes, access was restricted to the private IP address of the Web Server (172.31.46.164/32) instead of allowing connections from the public internet. This configuration ensured that only the authorized web server could establish a connection to the database server, protecting the database from unauthorized access.

<img width="383" height="574" alt="image" src="https://github.com/user-attachments/assets/a3f3bab9-b1fd-4d55-ad99-adf122bbcb52" />

The wp-config.php file was edited to configure WordPress to connect to the remote MariaDB database server. The default database configuration values were replaced with the actual database name, username, password, and the private IP address of the database server. The database name was set to wordpress, the database user to myuser, the password to mypass, and the database host to the private IP address 172.31.44.1. The database character set remained utf8mb4, which supports a wide range of Unicode characters, while the default collation setting was retained. These settings enable WordPress to establish a secure connection to the remote database during the installation and normal operation of the website.

<img width="554" height="469" alt="image" src="https://github.com/user-attachments/assets/d3f5b501-7a56-42fd-abe0-169837f3b1b1" />
<img width="554" height="440" alt="image" src="https://github.com/user-attachments/assets/dfd7317c-757a-4ad2-ab9a-e07d45104220" />
<img width="434" height="498" alt="image" src="https://github.com/user-attachments/assets/1837473d-852d-4b19-ba93-8b9d4dae1cb2" />
<img width="554" height="266" alt="image" src="https://github.com/user-attachments/assets/3ffc0ff1-4763-42ed-b154-3339ad43695e" />

After configuring the web server, connecting WordPress to the remote MariaDB database, and restarting the Apache service, the WordPress installation page was accessed through the web browser using the public IP address of the Web Server followed by /wordpress. The successful display of the WordPress five-minute installation wizard confirmed that Apache, PHP, the WordPress application, and the remote database connection were all configured correctly. The installation was completed by providing the site title, administrator username, password, and email address, after which WordPress was successfully deployed and ready for use.




































