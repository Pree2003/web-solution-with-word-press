# web-solution-with-word-press
Built a scalable WordPress web solution on AWS by deploying and configuring Apache, PHP, MySQL, and WordPress. 
Configured virtual hosts, database connectivity, and server security while demonstrating Linux administration and cloud deployment skills.
<img width="554" height="176" alt="image" src="https://github.com/user-attachments/assets/e7a2103c-ed9b-4275-bebd-5cc40fdee6f0" />
After launching the Red Hat EC2 instance and ensuring that the SSH port (22) was open in the Security Group, the instance was accessed remotely from a Windows machine using PowerShell and the downloaded private key (webserver.key.pem).

The following command was executed:
ssh -i "C:\Users\lenovo\Downloads\webserver.key.pem" ec2-user@13.60.173.0
Initially, the connection failed because the username Redhat was used:
ssh -i "C:\Users\lenovo\Downloads\webserver.key.pem" Redhat@13.60.173.0
This resulted in the following error:
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
The issue was resolved by using the correct default username for a Red Hat Enterprise Linux EC2 instance, which is ec2-user.
After correcting the username, the connection was established successfully, and the terminal displayed the EC2 instance prompt:
[ec2-user@ip-172-31-46-164 ~]$
This confirmed that the Red Hat server was successfully accessed via SSH using the private key without requiring a password.

