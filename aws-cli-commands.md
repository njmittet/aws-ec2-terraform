# AWS EC2 CLI

AWS CLI commands used to prepare for, create and delte an AWS EC2 instance.

```SH
# Displays the extensive CLI documentation.
$ aws ec2 help
```

## Key Pairs

An EC2 key pair is required for accessing the servers.

### Create a Key Pair

See [Controlling command output from the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-output.html) for the `--query` and `--output` options used.

```SH
# Creates the key pair and a file containing the private key.
$ aws ec2 create-key-pair --key-name ec2-ssh.eu-west-1 --query 'KeyMaterial' --output text > .ssh/ec2-ssh.eu-west-1
```

### List All Key Pairs

```SH
$ aws ec2 describe-key-pairs
{
    "KeyPairs": [
        {
            "KeyPairId": "key-0f9624ec81323293e",
            "KeyFingerprint": "1d:97:11:7e:c2:10:c8:97:ba:e7:26:4f:88:2c:47:6a:be:e2:19:8d",
            "KeyName": "ec2-ssh.eu-west-1",
            "Tags": []
        }
    ]
}
```

### Describe a Key Pair

```SH
$ aws ec2 describe-key-pairs --key-name ec2-kp.eu-west-1
{
    "KeyPairs": [
        {
            "KeyPairId": "key-0f9624ec81323293e",
            "KeyFingerprint": "1d:97:11:7e:c2:10:c8:97:ba:e7:26:4f:88:2c:47:6a:be:e2:19:8d",
            "KeyName": "ec2-ssh.eu-west-1",
            "Tags": []
        }
    ]
}

```

### Delete a Key Pair

```SH
# Delete a key pair by name. Other identifiers like --key-pair-id may be used.
$ aws ec2 delete-key-pair --key-name ec2-ssh.eu-west-1
```

## Security Groups

Security groups essentially operates as a firewall by providing rules that determine what network traffic can enter and leave the server using the securty group. An EC2 instance my beloing to multiple security groups.

### List Security Groups

```SH
# List all security groups. Complete config is returned.
$ aws ec2 describe-security-groups
```

### Describe a Security Group

```SH
# Security groups can only be references by the group id, not the name.
$ aws ec2 describe-security-groups --group-ids <security-group-id>
{
    "SecurityGroups": [
        {
            "Description": "Allow SSH access",
            "GroupName": "ec2-sg.eu-west-1",
            "IpPermissions": [
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0",
                            "Description": ""
                        }
                    ],
                    "Ipv6Ranges": [
                        {
                            "CidrIpv6": "::/0",
                            "Description": ""
                        }
                    ],
                    "PrefixListIds": [],
                    "ToPort": 22,
                    "UserIdGroupPairs": []
                }
            ],
            "OwnerId": "882837571239",
            "GroupId": "<security-group-id>",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-3318b855"
        }
    ]
}
```

### Create a Security Group

Creating a security group is a two-step process. First, create the actual security group:

```SH
$ aws ec2 create-security-group --group-name ec2-sg.eu-west-1.web --description 'Web access' --vpc-id <vpc-id>
{
    "GroupId": "<security-group-id>"
}
```

Then add rules to the security group. See [authorize-security-group-ingress](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/authorize-security-group-ingress.html) for details:

```SH
# The rule below allows port 80 access from anywhere.
$ aws ec2 authorize-security-group-ingress --group-id <security-group-id> --protocol tcp --port 80 --cidr '0.0.0.0/0'

# Adding the same rule for IPv6 addresses requires usage of the --ip-permissions option. This is the shorthand version.
$ aws ec2 authorize-security-group-ingress --group-id <security-group-id> --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,Ipv6Ranges='[{CidrIpv6=::/0}]'

# The --ip-permissions options also accepts JSON, here with rules for both IPv4 and IPv6.
$ aws ec2 authorize-security-group-ingress --group-id <security-group-id> --ip-permissions \
'[
  {
    "FromPort": 443,
    "ToPort": 443,
    "IpProtocol": "TCP",
    "IpRanges": [
      {
        "CidrIp": "0.0.0.0/0"
      }
    ],
    "Ipv6Ranges": [
      {
        "CidrIpv6": "::/0"
      }
    ]
  }
]'
```

### Delete a Security group

Deleting a security group is basically a reversal of the creation process. First, delete the rules, which is similar to adding rules in terms of how options must be used:

```SH
# The rule below removes port 80 access from anywhere.
$ aws ec2 revoke-security-group-ingress --group-id <security-group-id> --protocol tcp --port 80 --cidr '0.0.0.0/0'

# Removing the same rule for an IPv6 address requires usage of the --ip-permissions option. This is the shorthand version.
$ aws ec2 revoke-security-group-ingress --group-id <security-group-id> --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,Ipv6Ranges='[{CidrIpv6=::/0}]'

# Using JSON, here with rules for both IPv4 and IPv6.
$ aws ec2 revoke-security-group-ingress --group-id <security-group-id> --ip-permissions \
'[
  {
    "FromPort": 443,
    "ToPort": 443,
    "IpProtocol": "TCP",
    "IpRanges": [
      {
        "CidrIp": "0.0.0.0/0"
      }
    ],
    "Ipv6Ranges": [
      {
        "CidrIpv6": "::/0"
      }
    ]
  }
]'
```

See [revoke-security-group-ingress](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/revoke-security-group-ingress.html) for details:

Then, delete the actual security group:

```SH
# A security group can't be deleted if the group is currently attached to an environment.
$ aws ec2 delete-security-group --group-id <security-group-id>
```

## EC2 Instances

Using the above key pairs and security groups to create an EC2 instance.

### Create Instance

See [run-instances](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/run-instances.html) for details.

```SH
# ami-063d4ab14480ac177 is Amazon Linux.
$ aws ec2 run-instances --image-id  ami-063d4ab14480ac177 --count 1 --instance-type t2.micro --key-name ec2-kp.eu-west-1 --security-group-ids <security-group-id>
```

### Add Instance Storage

Use block device mapping to specify additional Amazon Elastic Block Store (Amazon EBS) volumes or instance store volumes. To add a block device to an instance, specify the --block-device-mappings option when creating the instance. See [Block Device Mapping](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/block-device-mapping-concepts.html) and [Preserve Amazon EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html#preserving-volumes-on-termination) for details.

```SH
# Provisions a standard Amazon EBS volume that is 10 GB in size, and maps it to your instance using the identifier /dev/sdf. DeleteOnTermination is false by default.
$ aws ec2 run-instances --image-id ami-063d4ab14480ac177 --count 1 --instance-type t2.micro --key-name ec2-kp.eu-west-1 --security-group-ids <security-group-id> --block-device-mappings '[{"DeviceName":"/dev/sdf\","Ebs":{"VolumeSize":10, "DeleteOnTermination":false}}]'
```

The above EBS block storage is not deleted when the instance is terminated. The preserved volume can be attached to another running instance using [attach-volume](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/attach-volume.html) or deleted when not attached to any instances by using [delete-volume](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/delete-volume.html). Use [describe-volums](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-volumes.html) to list all volumes.

If a snapshot (copy) of the volume is created, the snapshot can be used when creating new instances by using the snapshot id. See [create-snapshot](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/create-snapshot.html) and [delete-snapshot](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/delete-snapshot.html). List own snapshots with [describe-snapshots](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-snapshots.html).

```SH
# The following example adds an Amazon EBS volume, mapped to /dev/sdf, based on an existing snapshot.
$ aws ec2 run-instances --image-id ami-063d4ab14480ac177 --count 1 --instance-type t2.micro --key-name ec2-kp.eu-west-1 --security-group-ids <security-group-id> --block-device-mappings '[{"DeviceName":"/dev/sdf","Ebs":{"SnapshotId":"snap-0f43df9d631e0cba1", "DeleteOnTermination":true}}]'
```

### Add Instance Tag

A tag enables adding metadata to a resources that you can use for a variety of purposes. See [create-tags](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-tags.html) for details.

```SH
# Add a single tag to a specified instance.
$ aws ec2 create-tags --resources <instance-id> --tags Key=Name,Value=AwsCertification
```

### List Instances

Use query and filering options to limit output. See [Controlling command output from the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-output.html) for the `--query` and `--output` options used.

See [describe-instances](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-instances.html) for details.

Describe a single instance:

```SH
# Show the InstanceId, Tags and BlockDeviceMapping of a specified instance.
$ aws ec2 describe-instances --instance-id <instance-id> --query 'Reservations[*].Instances[*].[InstanceId,BlockDeviceMappings,Tags[*].Value]'
[
    [
        [
            "i-05f452d940eded227",
            [
                {
                    "DeviceName": "/dev/xvda",
                    "Ebs": {
                        "AttachTime": "2021-05-06T15:51:18+00:00",
                        "DeleteOnTermination": true,
                        "Status": "attached",
                        "VolumeId": "vol-0a302765080d0b3d7"
                    }
                }
            ],
            [
                "AwsCertification"
            ]
        ]
    ]
]
```

Describe a tagged instance:

```SH
# Output the InstanceId as text.
$ aws ec2 describe-instances --filters "Name=tag:Name,Values=AwsCertification" --query 'Reservations[*].Instances[*].[InstanceId]' --output text
i-05f452d940eded227
```

Describe multiple instances of a given type:

```SH
# Show the InstanceId, LaunchTime and security group GroupName for all t2.micro instances.
$ aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro"  --query 'Reservations[*].Instances[*].[InstanceId,ImageId,LaunchTime,SecurityGroups[*].GroupName]'
[
    [
        [
            "i-05f452d940eded227",
            "ami-063d4ab14480ac177",
            "2021-05-06T15:51:17+00:00",
            [
                "ec2-sg.eu-west-1.ssh"
            ]
        ]
    ],
    [
        [
            "i-05c4a21fd6c2f5f68",
            "ami-063d4ab14480ac177",
            "2021-05-10T17:24:02+00:00",
            [
                "ec2-sg.eu-west-1.ssh",
                "ec2-sg.eu-west-1.web"
            ]
        ]
    ]
]

```

### Terminate Instance

Terminating an instances deletes the instance. See [terminate-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html) for details.

```SH
# Terminating one or more instances.
$ aws ec2 terminate-instances --instance-ids <instance-id>
```
