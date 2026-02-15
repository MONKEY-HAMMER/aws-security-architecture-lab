# EC2 Storage

### Amazon Elastic Block Store (EBS)

Generally, in EC2 instances, the persistent storage used is EBS (Elastic Block Store). 

#### Snapshots 
EBS snapshots are incremental, block-level backups of EBS volumes stored internally in S3. They are used to create new volumes, AMIs, and support disaster recovery strategies.


```bash
aws ec2 describe-snapshots --owner-ids self
``` 
`--owner-ids self`: It means that only snapshots that belong to you will be displayed.


#### Snapshots are regional.
##### If you are configured for:
`us-east-1`

Then, only snapshots from the current region (us-east-1) will be displayed.

--- 
```bash
aws ec2 describe-snapshots --snapshot-ids snap-0abc1234def567890
``` 
This command returns detailed information about a specific snapshot, identified by the SnapshotId.
```json 
{
  "Snapshots": [
    {
      "SnapshotId": "snap-0abc1234def567890",
      "VolumeId": "vol-0123456789abcdef0",
      "State": "completed",
      "StartTime": "2026-02-15T18:22:30.000Z",
      "Progress": "100%",
      "VolumeSize": 8,
      "Encrypted": false,
      "OwnerId": "123456789012",
      "Description": "Backup of production volume"
    }
  ]
}
```

--- 

## How to restore a volume from a snapshot
As we know, snapshots are regional, this means that a Volume must be in the same  Availability Zone (AZ) as the machine you will mount it from. 

#### Command to get the AZ of your EC2 Instance 
```bash
TOKEN=`curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
```

```bash
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/availability-zone
```


Now you can create a volume from the snapshot.

```bash
aws ec2 create-volume --snapshot-id snap-... --volume-type gp3 --region us-east-1 --availability-zone YOUR-EC2-AZ
```

```json
{
    "AvailabilityZone": "us-east-1a",
    "MultiAttachEnabled": false,
    "Tags": [],
    "Encrypted": false,
    "VolumeType": "gp3",
    "VolumeId": "vol-...",
    "State": "creating",
    "Iops": 3000,
    "SnapshotId": "snap-...",
    "CreateTime": "2026-02-14T20:21:04.000Z",
    "Size": 1
}
```

Attaching the EBS volume created to this current EC2 instance:

- You will need the Instance Id
```bash
instance_id=$( curl  -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/instance-id )
``` 


```bash
aws ec2 attach-volume --region us-east-1 --device /dev/sdh --instance-id $instance_id --volume-id vol-YOUR-VolumeId
```
You have now successfully restored a volume from a snapshot. You can confirm with:

```bash
sudo fdisk -l
``` 