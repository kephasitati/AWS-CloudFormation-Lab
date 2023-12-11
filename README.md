# AWS CloudFormation Lab

## Task 1: Deploying a Networking Layer

It is a best practice to deploy infrastructure in layers. Common layers are:

- Network (Amazon VPC)
- Database
- Application

This way, templates can be reused between systems. For example, you can deploy a common network topology between development, test, and production environments or deploy a standard database for multiple applications.

### Instructions:

1. Download the networking layer template to your computer: [lab-network.yaml](lab-network.yaml).
2. Open the template in a text editor to review AWS resources definitions.
3. In the AWS Management Console, navigate to CloudFormation.
4. Choose "Create stack" and configure settings:
    - Specify template: Upload a template file
    - Upload a template file: Choose the `lab-network.yaml` file
5. Review and configure stack options, providing tags.
6. Review and create the stack.

### Outputs:

- **PublicSubnet**: The subnet ID to use for public web servers (Exported as `${AWS::StackName}-SubnetID`).
- **VPC**: VPC ID (Exported as `${AWS::StackName}-VPCID`).

---

## Task 2: Deploying an Application Layer

Now that you deployed the network layer, you will deploy an application layer that contains an Amazon Elastic Compute Cloud (Amazon EC2) instance and a security group.

### Instructions:

1. Download the application layer template to your computer: [lab-application.yaml](lab-application.yaml).
2. Open the template in a text editor to review AWS resources definitions.
3. In the left navigation pane, choose Stacks.
4. Select "Create stack" > "With new resources (standard)" and configure settings:
    - Specify template: Upload a template file
    - Upload a template file: Choose the `lab-application.yaml` file
5. Configure stack options, providing tags.
6. Review and create the stack.

### Outputs:

Copy the URL displayed in the Outputs tab, open a new web browser tab, paste the URL, and press ENTER. The browser tab will open the application, which is running on the web server that this new CloudFormation stack created.

---

## Task 3: Updating a Stack

AWS CloudFormation can also update a stack that has been deployed. In this task, you will update the lab-application stack to modify a setting in the security group.

1. Examine the current settings for the security group in the EC2 console.
2. Download the updated application layer template to your computer: [lab-application2.yaml](lab-application2.yaml).
3. In the AWS Management Console, navigate to CloudFormation.
4. In the Stacks list, select `lab-application`.
5. Choose "Update" and configure settings:
    - Replace current template
    - Specify template: Upload a template file
    - Upload a template file: Choose the `lab-application2.yaml` file
6. Update the stack.

---

## Task 4: Exploring Templates with AWS CloudFormation Designer

AWS CloudFormation Designer is a graphic tool for creating, viewing, and modifying AWS CloudFormation templates. In this task, you will gain some hands-on experience with Designer.

1. In the AWS Management Console, navigate to CloudFormation.
2. In the left navigation pane, choose Designer.
3. Choose the File menu, select Open > Local file, and select the `lab-application2.yaml` template.
4. Experiment with the Designer features to understand the interrelationship between the template's resources.

---

## Task 5: Deleting the Stack

When resources are no longer required, AWS CloudFormation can delete the resources built for the stack. A deletion policy can also be specified against resources. It can preserve or back up a resource when its stack is deleted.

1. In the main AWS CloudFormation console, select `lab-application`.
2. Choose "Delete stack" and confirm the deletion.
3. Monitor the deletion process in the Events tab.
4. Verify that a snapshot of the EBS volume was created before the EBS volume was deleted in the EC2 console.

---

## lab-network.yaml

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: >-
  Network Template: Sample template that creates a VPC with DNS and public IPs enabled.

# This template creates:
#   VPC
#   Internet Gateway
#   Public Route Table
#   Public Subnet

######################
# Resources section
######################

Resources:

  ## VPC

  VPC:
    Type: AWS::EC2::VPC
    Properties:
      EnableDnsSupport: true
      EnableDnsHostnames: true
      CidrBlock: 10.0.0.0/16
      
  ## Internet Gateway

  InternetGateway:
    Type: AWS::EC2::InternetGateway
  
  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  
  ## Public Route Table

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
  
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  
  ## Public Subnet
  
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.0.0/24
      AvailabilityZone: !Select 
        - 0
        - !GetAZs 
          Ref: AWS::Region
  
  PublicSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable
  
  PublicSubnetNetworkAclAssociation:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      NetworkAclId: !GetAtt 
        - VPC
        - DefaultNetworkAcl
  
######################
# Outputs section
######################

Outputs:
  
  PublicSubnet:
    Description: The subnet ID to use for public web servers
    Value: !Ref PublicSubnet
    Export:
      Name: !Sub '${AWS::StackName}-SubnetID'

  VPC:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VPCID'


