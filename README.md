# AWS EC2

## Resouces

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html](EC2 Getting Started)  
[https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2.html](EC2 CLI Commands)

## EC2 Instance Users

Add a user with SSH access:

```
$ sudo adduser new_user
$ mkdir .ssh
$ chmod 700 .ssh
$ touch .ssh/authorized_keys
$ chmod 600 .ssh/authorized_keys
```

Then add the public part of a key pair to `authorized_keys`

Delete a user:

```
$ sudo userdel -r olduser
```

Add a user to the sudoers group:

```
# Add a password to the user, otherwise just logging in using the SSH certificate.
$ sudo passwd new_user

# For Amazon Linux, Amazon Linux 2, RHEL, and CentOS:
$ sudo usermod -aG wheel new_user
```