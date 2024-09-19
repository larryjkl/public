# Spot NAT Instances — A cheaper AWS NAT Gateway alternative

## Introduction and Overview

Since I started using AWS, I have been looking for a reasonable replacement for the NAT gateway service. If an organization has many individual VPCs (like we do), especially for sandbox and test accounts (or personal accounts), the NAT gateway costs can be significant.

As of June 2024, the cost of a NAT gateway is around $32 a month. There is also an extra $0.045 per GB charge, which adds a 50% premium to egress bandwidth charges.

A NAT gateway has advantages over using a single NAT instance (which AWS also offered but has now been deprecated). Some of the more significant ones are:

- Highly available. NAT gateways are implemented with redundancy.
- Scales as needed. NAT gateways can scale up to 100Gbs if necessary.
- AWS Managed. The service is managed entirely by AWS.

My goal was to find a more cost effective solution that addresses some of the challenges of standing up a single NAT instance. There are a number of other solutions that have been built; that have pursued this and/or solved it in various ways; I’ll link to those sites at the bottom.

I started with the following goals::

- A simple solution written in CloudFormation
- No user interaction needed in the case of a NAT instance failure
- Use spot instances to reduce the cost even further
- Even when the NAT instance is being replaced, devices using the NAT instance shouldn’t lose connectivity to the internet

You will need to have a VPC created that has at least one public subnet ( which is where the NAT Instance will be created ). The script will also create a route table that can be attached to private or private-nat subnets so that they can utilize the NAT instance

## Template Breakdown

The CloudFormation template creates a number of resources while building the solution:

- Security Group — For the NAT instance to use. It allows traffic only from the CIDR range of the VPC
- IAM Role — For the NAT instance to use. Just has the permissions needed to manage being a NAT instance
- Launch Template — Defines some details of the NAT instance; specifically it has the startup script that the instance uses to become a
- NAT instance and update the route table so that it becomes the primary NAT instance
- Auto Scaling Group — This allows that NAT instance(s) to wait until a new NAT instance is created before terminating and defines multiple types of instances that can be used ( to help ensure there is a spot instance available )
- Route Table — This will be updated automatically by the latest NAT instance when it starts so that it always has a route to the internet. This is the route table that should be used by subnets wanting to reach the internet

The parameters required to the run the CloudFormation script are:

- Application : This is an id that you can use to identify the resources that are created by the script. I add this in most of my scripts; it makes it easier to identify resources that the application or business functionality that they were created for. But leaving it as default is fine
- VpcCidr: This is the CIDR range of the VPC that you are creating the NAT instance for. It’s used when creating the security group to restrict the set of IP addresses that are able to access the NAT instance ( ex. 192.168.100.0/24 )
- LatestAmiId : This defaults to the x64 Linux 2 AMI that AWS has published and can be left alone
- VpcId: This is the VPC id that the NAT instance will be used in
- PublicSubnetId : The subnet id of a public subnet where the NAT instance will be created
- EipAlloc1 and EipAlloc2 : If you want your NAT instance to always have well a set of well defined public IP addresses ( for IP whitelisting for example ) you can put two Elastic IP address allocation IDs in these parameters and whenever a NAT instance is created it will use one of the two IP addresses ( normally whichever one is not being used )

## Deployment Instructions

After running the CloudFormation script you should see new resources created including a new ec2 instance ( the NAT instance ) and a new route table ( called [application]-natinst-rt ). In order to allow resources in a private or private-with-nat subnet to use the NAT instance and reach the internet, associate those subnets with the new route table.

### NAT Instance replacement process

There are a number of reasons why the NAT instance could be terminated, especially using spot instances. The autoscaling group and the NAT instances are configured to reduce or eliminate any network connectivity downtime during replacements.

The autoscaling group is configured to watch for spot rebalance notifications and automatically start a new spot instance before the previous one is taken away. There is also a lifecycle hook configured to pause for 60 seconds before terminating an instance. This allows the new instance that is created to become a NAT instance and update the route table to point to itself so there is little to no connectivity interruption for the resources using the NAT instance. The only disruption will be to existing active connections out to the internet and the outgoing IP address of the traffic will change to a new IP address.

When a new instance is launched is goes through a few steps to become the new NAT instance:

- Updates the new instance with any security patches that are available
- If there were EIPs specified to be use for the public IP address, attempts to associate an address that is not being used with the new instance
- Modifies the settings of the instance to act like a NAT instance
- Updates the natinst route table to send all internet traffic to itself

### Bandwidth restrictions

While most AWS instances allow you to burst up to 5Gbs of traffic, their baseline ( or sustained ) bandwidth is much smaller. This is one the challenges of using a NAT instance instead of a NAT gateway. Another NAT solution has a great deal of information on instances and bandwidth at Choosing an Instance Size — fck-nat . The instances that are currently in the CloudFormation template are some of the smallest. Currently the template is configured to only use x86-type instances, its easy to modify the instance types used to suit your use case. But, since it is using spot instances, having more than one instance type ( ideally 3 or more ) ensures high spot availability.

The existing script has these parameters for driving the spot instance choices:


```
    Overrides:
      - InstanceType: t2.micro           
      - InstanceType: t3.micro           
      - InstanceType: t3a.micro           
      - InstanceType: t2.small
      - InstanceType: t3.small
      - InstanceType: t3a.small
  InstancesDistribution:
    SpotAllocationStrategy: capacity-optimized-prioritized
```

Depending on your requirements this would be the place to customize the instance types to be used and/or the spot allocation strategy

## Conclusion and Next Steps

Thanks for getting this far : ). Please reach out via the comments or GitHub issues if there are any questions, issues or suggestions.

There are a number of other NAT instance solutions out there, each with different advantages and challenges. But I believe this solution which provides very reliable NAT functionality at spot prices seems fairly unique. But here are links to other solid alternative solutions that I found during my research and development.

[](https://medium.com/@larryjkl/spot-nat-instance-cloudformation-template-for-aws-e0e9f13719a5)
