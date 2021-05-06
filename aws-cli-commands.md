# AWS EC2 CLI

## EC2

```
aws ec2 help
```

Create a key pair. See [Client Side Filtering](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-filter.html#cli-usage-filter-client-side) for the `--query` and `--output` options used. 

```
$ aws ec2 create-key-pair --key-name ec2-kp.eu-west-1 --query 'KeyMaterial' --output text > .ssh/ec2-kp.eu-west-1
```

List all key pairs

```
$ aws ec2 describe-key-pairs
```

List a key pairs

```
$ aws ec2 describe-key-pairs --key-name ec2-kp.eu-west-1
```

Delete a key pair

```
aws ec2 delete-key-pair --key-name ec2-kp.eu-west-1
```


## Security Groups

``` 
```

## SSH Key Pairs

Create key pair

```
```

Display key pair

```
```

Delete key pair