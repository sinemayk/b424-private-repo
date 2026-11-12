# Troubleshooting AWS Network Connectivity

## Introduction
Troubleshooting basic network connectivity issues is an important skill. This troubleshooting scenario is an opportunity to assess your skills in this area.

In this lab scenario, a junior administrator has deployed a VPC and instances, but there are a few things wrong. Instance3 is not able to connect to the internet and the junior admin can't determine why. Being a senior administrator, it's your responsibility to troubleshoot the issue and ensure the instance has connectivity to the internet, so that you can ping and log in to the instance using SSH.

## Start to find a Solution
- Log in to the AWS Management Console. 
Make sure you're using the us-east-1 region.

- Determine Why the Instance Cannot Connect to the Internet
Navigate to the EC2 dashboard.
Click Instances (running).
Select Instance3 from the list and review the instance details. 
* (NOTE: Notice Instance3 does not have a public IP address.)
At the top of the page, click Actions, to select Networking, and select Manage IP addresses.
In the tooltip box, click allocate to be redirected to the Elastic IP addresses list.
In the top-right corner, click Allocate Elastic IP address.
Leave the settings as default and click Allocate.
Select the IP address and click the Actions dropdown menu to select Associate Elastic IP address.
For Instance, select Instance3 and click Associate.
In the left-hand menu, click Instances.
Select Instance3 from the list and review the instance details. (NOTE: Notice Instance3 now has a public IP address.)
Copy the public IP address of Instance 3 and attempt to ping the instance from either your own local terminal or ssh into Instance 1. to do the ping test:
ping <Instance3_Public_IP_Address>
- Answer the question: Can you ping?

## Identify the Issues Preventing the Instance from Connecting to the Internet
- Navigate back to AWS Instances page.
- Select Instance3 from the list of instances.
- Click Security tab to review the associated security group.
- Review the security group details.
- In the left-hand menu, click Security Groups.
- Scroll to Security group name and expand the field to locate EC2SecurityGroup3.
- Click Inbound rules tab to view the allow and deny inbound rules.
- Click Outbound rules tab to view the allow and deny outbound rules.
- In the left-hand menu, click Instances.
- Select Instance3 and click Networking tab to view the private subnet IP address.
- In a new tab, navigate to VPC.
- In the left-hand menu, click Subnets to find the private subnet IP address that matches Instance3.
- Select PublicSubnet4 and click Network ACL tab.
- Click on the link next to Network ACL.
- Click Inbound rules tab to view the allow and deny inbound rules.
- Click Outbound rules tab to view the allow and deny outbound rules.
- Click the "X" on the Network ACL ID or Clear filters to view all available Network ACLs.
Select Public3-NACL.
Click Inbound rules tab to view the allow and deny inbound rules.
Click Outbound rules tab to view the allow and deny outbound rules.
In the left-hand menu, click Subnets and select PublicSubnet4.
Click Network ACL tab and click Edit network ACL association.
For Network ACL ID, select Public3-NACL from the dropdown menu.
Click Save.
Navigate back to the terminal to verify ping results.
Navigate back to the AWS Subnets page and click Route table tab to view the associated route table, Private3-RT.
Click Edit route table association.
For Route table ID, select Public3-RT from the dropdown menu.
Click Save.
In the left-hand menu, click Route Tables.
Select Public3-RT and click Routes tab to view the routes attached.
Navigate back to the terminal to confirm successful ping results. (NOTE: Feel free to try to SSH using the provided credentials.)
Conclusion
Congratulations — you've completed this hands-on lab!