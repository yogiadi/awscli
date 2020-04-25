# Analyze Big Data With Hadoop

## Create an S3 bucket

```console

aws s3 mb s3://yogiadibucket
aws s3api create-bucket --bucket yogiadibucket --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1

```

## Create a folder in S3 bucket

```console 

aws s3api put-object --bucket yogiadibucket --key MyHiveQueryResults

```

## Create Key Value Pair

```console

aws ec2 create-key-pair --key-name MyKeyPair

```

```json
{
    "KeyFingerprint": "d7:a5:62:4e:8d:a6:f9:e9:1b:01:74:3e:87:7f:da:15:f7:b8:d3:df",
    "KeyMaterial": "-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEAlOtPyW8nzMa4BhKtAKMV82jfxLZq4kaPflD/y+gyH+dEt5vTzYaumG2o6eIf\nxFnGeDTmZxKVqqIfMD6Xv7APAH3hGWWksY+2EQb6+YYl0pH7CqQw8lKMMws0VgcPHexvgyo7JM+I\nLkU8eV4dBz9d6UAwcMDMgLkhKqzG4FWijIhCJAbzqjf6J88O5zg52AfkG3x8DuEZY8Mm7HsVW0tZ\nsb/Fghd2NvN8Tp3R5BoEH0U+G1SKhmRS185yYN9ObZH0DgYzpSBnjdnFOGHpqjKETw8Ct5Pv+pEJ\nEk1KAKEWH8SCFfSp0bQFkABq+xiJ1EbxMssPrQprYluXRvRV/hRhywIDAQABAoIBAE6kSXk1sw3n\n7yx91lCczy3At1LZhm5CFur8+WiEEVxZtCdGKj7CHheu6WHBoUb+pdm7DeVmohT/EntWwqpe3j1D\nPIk97RA7tUkep5D376dYofsHpDWDtDgMdbHsfmYuAuGAfsU8t0zAEWSCd8/o/b38wNf6aMSdf09a\n+Jdlgba2OyvQYBnuXJ/eZ030eX84R+VBG6yrrQTXxWp9GFeOjHD02OgFSO+IrAD69o3wvi2eO8B7\niTp1U7GfeGyGuGCRqWmJgF9VlXDMt9BKcnTntkKRQBUHLvZD/0TwtZtP/IxYtVxZNZXebJJ7asT1\nDHyWdcHVq1oWpYQMFCBZGgJuLNkCgYEA1du6jC6EihFtmT072XC/Q/aZAUJhqJTm2DJdmSueSl4S\naUzEHOT62954p0j/ryOikqQ51WA871pXwGuLEgbe7QtNz4dp9EY1xxjHYwx6scXVIN1bJLpWUi0K\nmKfhVuPi/iGfpnpRnDbwl7lC9/AQXyPF0dly/XKST3Ji4KUaEt8CgYEAskOr3LMuimlOr1mVPOw1\nPqnc7TGeVwLa5nNhzecPrto9ZN7+pahiigryuvM4XNIuF3KVKm9Er8wOAZYQI/8gmwG46rvb8mwz\n0FgGgCQkGs1ICP6vy7D+5iPgJ4MmsshNWZO5Xhf0rMlLnNe4KfH6rroZxiOFzFD0h3K2cLpbWpUC\ngYBna2wqUHCeCFGVEoy32ZNVPZK+jiiGCwQbtzeWVAandhi3PX11b7J004BLfbzQKI59Fdg1/OVa\n5ribACbv38y8m4PUgWrWy4FEOBJsLuPCxqYLz6A9AbOu58NBEaaii3ABb6cyLeL7hYISDXB6UGjK\naZVHpz25nUduDiYM9yrFxwKBgHRcfeObRqbspIMLa8IPwZW81UwzFlNftrm02UPJLtCI/ohIk1pY\n9rF7AgSMXN8iBxohHkNLzD8gaIgah0Cn+YWU8zquE51DifLWcq/UZ4jjNMFCVkgUqd5ZzqicEHel\nDfCJ3/cOlhTvdJ7VpQ4kOOky6z4N0/mRYnzDoVkmHBmdAoGBAIEc2//AgejzxkqSZSPmgxM/e244\nzPN80BCkXUmvlChGtBjufVz7df0GyGq8Dgk2u3VHRKc8uhHtO0glalq+Lb0T8+w5ZkZ5AbTgX4vg\nuEaW35RW5btAEFcOM699uPExftb6tF26fxnhcLeA2DzQPs/R4pxYig85vTFyy1xHy/MH\n-----END RSA PRIVATE KEY-----",
    "KeyName": "MyKeyPair",
    "KeyPairId": "key-059e4bd41a68037dc"
}

```

## Create an EMR Cluster

```console

aws emr create-cluster --applications Name=Ganglia Name=Hadoop Name=Hive Name=Hue Name=Mahout Name=Pig Name=Tez --ec2-attributes '{"KeyName":"MyKeyPair","InstanceProfile":"EMR_EC2_DefaultRole","SubnetId":"subnet-0d761541","EmrManagedSlaveSecurityGroup":"sg-0c7f038eb88881fa7","EmrManagedMasterSecurityGroup":"sg-0eed12a66ad5c7dba"}' --service-role EMR_DefaultRole --enable-debugging --release-label emr-5.29.0 --log-uri 's3n://aws-logs-390993126262-ap-south-1/elasticmapreduce/' --name 'My cluster' --instance-groups '[{"InstanceCount":2,"EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"SizeInGB":32,"VolumeType":"gp2"},"VolumesPerInstance":2}]},"InstanceGroupType":"CORE","InstanceType":"m5.xlarge","Name":"Core Instance Group"},{"InstanceCount":1,"EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"SizeInGB":32,"VolumeType":"gp2"},"VolumesPerInstance":2}]},"InstanceGroupType":"MASTER","InstanceType":"m5.xlarge","Name":"Master Instance Group"}]' --scale-down-behavior TERMINATE_AT_TASK_COMPLETION --region ap-south-1

```
