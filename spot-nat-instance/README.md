# Spot NAT Instance CloudFormation template for AWS

Features
- Single CloudFormation script
- Autoscaling group should automatically replace NAT instance on existing instance termination
- NAT instance makes updates to route table automatically on launch
- Resources using NAT instance shouldn't lose connectivity with instance refreshes or spot terminations
- Uses Spot instances to reduce costs

Drawbacks
- Limited to a single instance, so bandwidth can't scale past the instances capacity
- Network interface changes on instance relaunch making VPC Flow log capture more difficult ( Potentially solvable )
- Source public IP address changes on instance refresh ( Potentially solvable )
- Non-standard termination of NAT instance will cause connectivity loss until autoscaling group creates replacement instance
- Only works for VPCs with a single CIDR range ( Potentially solvable )
