# Spot NAT Instance CloudFormation template for AWS

## Features
- Single CloudFormation script
- Autoscaling group should automatically replace NAT instance on existing instance termination
- NAT instance makes updates to route table automatically on launch
- Resources using NAT instance shouldn't lose connectivity with instance refreshes or spot terminations
- Uses Spot instances to reduce costs

## Drawbacks
- Limited to a single instance, so bandwidth can't scale past the instances capacity
- Network interface changes on instance relaunch making VPC Flow log capture more difficult ( Potentially solvable )
- Source public IP address changes on instance refresh ( Potentially solvable )
- Non-standard termination of NAT instance will cause connectivity loss until autoscaling group creates replacement instance
- Only works for VPCs with a single CIDR range ( Potentially solvable )

## Cloudformation parameters
- **Application**: This id will get added to the resources created to tell them apartfrom other stacks that get created
- **VPCId**: The VPC id that will be used to create the NAT instance and route table
- **VPCCIDR**: The CIDR range of the VPC
- **LatestAmiId**: This will default to the lateast AWS Linux2 x86 image which is the starting point for the NAT Instance
- **PublicSubnetId**: The public subnet ID that the NAT instance will get created in

After the CloudFormation stack is finished; one of the resources that is created is a route table. To use the NAT instance,
attach this route table to the private-nat or private subnets that you want to have access to the internet.

Medium article: 

https://medium.com/@larryjkl/spot-nat-instance-cloudformation-template-for-aws-e0e9f13719a5
