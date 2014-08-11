Connecting to Amazon EC2 server on Mac using Terminal app
Posted on April 24, 2013
Go to the folder where your .pem file is stored. You can retrieve the .pem file from your amazon account.

Launch the terminal app and run the  command

cd /folder/where/perm/file/stored/
ssh-add filename.pem

This should work fine most of the time and you should get a response similar to this.

Identity added:xxxxx

If the permission set on the .pem file aren’t correct then terminal will show this error

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@ WARNING: UNPROTECTED PRIVATE KEY FILE! @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0444 for 'filename.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
ec2-user@xx.xx.xx.xx:22: No such file or directory

To fix it, set the appropriate permissions.

chmod 400 filename.pem

Now, it’s time to connect to ec2 server.

ssh-add filename.pem
ssh ec2-user@xx.xx.xx.xx:22

If evrything works fine, then you should see this response on your terminal screen:

Last login: Tue Apr 23 03:34:27 2013 from xx.xx.xx.xx
__| __|_ )
_| ( / Amazon Linux AMI
___|\___|___|
https://aws.amazon.com/amazon-linux-ami/2012.09-release-notes/

There are 15 security update(s) out of 141 total update(s) available
Run "sudo yum update" to apply all updates.
Amazon Linux version 2013.03 is available.
-bash: warning: setlocale: LC_CTYPE: cannot change locale (UTF-8)
[ec2-user@ip-xx-xx-xx-xx ~]$
