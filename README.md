# AWS EC2 Terraform

Project used to learn AWS EC2, using both the CLI API and Terraform.

## Resouces

[EC2 Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
[EC2 CLI Commands](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2.htm)

## EC2 Instance Users

Add a user. For AWS Linux `--disabled-password` is default does not need to be provided.

```SH
sudo adduser username

# Prepare SSH login for the user.
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```

Then add the public part of a key pair to `.ssh/authorized_keys`.

Delete a user:

```SH
# Deletes a single user. The -r flag ensures deletion of the users home directory.
sudo userdel -r username
```

Add a user to the sudoers group:

```SH
# A password is required for a user to be able to sudo.
sudo passwd newuser

# Add the user to the sudoers list. For Amazon Linux, Amazon Linux 2, RHEL, and CentOS:
sudo usermod -aG wheel username
```
