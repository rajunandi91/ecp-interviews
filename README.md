
# Python
## Explain what this code is doing and how the output will look like.Also update the code so that it can work for multi regions along with error handling.

```py
import boto3

def list_prod_subnets(vpc_id, region='us-east-1'):
    ec2 = boto3.client('ec2', region_name=region)
    subnets = ec2.describe_subnets(
        Filters=[
            {'Name': 'vpc-id', 'Values': [vpc_id]},
            {'Name': 'tag:Environment', 'Values': ['Prod']}
        ]
    )['Subnets']

    for subnet in subnets:
        print(f"Subnet ID: {subnet['SubnetId']}, AZ: {subnet['AvailabilityZone']}")

```

## Evaluation:-

- This Python function uses Boto3 to interact with AWS EC2 Service/VPC
- It filters subnets within the specified vpc_id that have a tag named Environment with value Prod
- It then prints the Subnet ID and Availability Zone of each matching subnet.
- Expected Output Format

```
Subnet ID: subnet-0abc1234def567890, AZ: us-east-1a
Subnet ID: subnet-0def4567abc890123, AZ: us-east-1b
```
- Sample code for multi-region

```py
import boto3
from botocore.exceptions import ClientError

def list_prod_subnets_multi_region(vpc_id):
    session = boto3.Session()
    regions = [region['RegionName'] for region in session.client('ec2').describe_regions()['Regions']]
    
    for region in regions:
        ec2 = session.client('ec2', region_name=region)
        try:
            subnets = ec2.describe_subnets(
                Filters=[
                    {'Name': 'vpc-id', 'Values': [vpc_id]},
                    {'Name': 'tag:Environment', 'Values': ['Prod']}
                ]
            )['Subnets']
            
            if subnets:
                print(f"\nRegion: {region}")
                for subnet in subnets:
                    print(f"  Subnet ID: {subnet['SubnetId']}, AZ: {subnet['AvailabilityZone']}")
        
        except ClientError as e:
            print(f"Error in region {region}: {e.response['Error']['Message']}")
```

# Data Structure + Python

## Build a simple log retention system that only keeps the most recent log per service in a dictionary format. 

Below is a list of logs in the format (service_name, timestamp)
```py
logs = [
    ("auth", 1682001000),
    ("api", 1682001010),
    ("auth", 1682001025),
    ("db", 1682000995),
    ("api", 1682001015),
    ("auth", 1682001011)
]
```
### Complete the Function
```py
def get_latest_logs(logs):
    latest = {}

    for service, ts in logs:
        # TODO: 

    return latest

```

## Evaluation
- Candidate uses a dict for constant-time lookups
- Correctly compares timestamps and keeps the latest
- Returns a dictionary, not a list
- Solution code
```py
def get_latest_logs(logs):
    latest = {}

    for service, ts in logs:
        if service not in latest or ts > latest[service]:
            latest[service] = ts

    return latest

```
- Expected Output
```py
{
    'auth': 1682001025,
    'api': 1682001015,
    'db': 1682000995
} 
```