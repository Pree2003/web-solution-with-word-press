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
