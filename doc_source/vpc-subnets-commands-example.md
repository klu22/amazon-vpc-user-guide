# Create an IPv4\-enabled VPC and subnets using the AWS CLI<a name="vpc-subnets-commands-example"></a>

The following example uses AWS CLI commands to create a nondefault VPC with an IPv4 CIDR block, and a public and private subnet in the VPC\. After you've created the VPC and subnets, you can launch an instance in the public subnet and connect to it\.

You will create the following AWS resources:
+ A VPC
+ Two subnets
+ An internet gateway
+ A route table
+ An EC2 instance

**Topics**
+ [Prerequisites](#vpc-subnets-commands-example-prereqs)
+ [Step 1: Create a VPC and subnets](#vpc-subnets-commands-example-create-vpc)
+ [Step 2: Make your subnet public](#vpc-subnets-commands-example-public-subnet)
+ [Step 3: Launch an instance into your subnet](#vpc-subnets-commands-example-launch-instance)
+ [Step 4: Clean up](#vpc-subnets-commands-example-clean-up)

## Prerequisites<a name="vpc-subnets-commands-example-prereqs"></a>

Before you begin, install and configure the AWS CLI\. When you configure the AWS CLI, you specify your AWS credentials\. The examples in this tutorial assume you configured a default Region\. Otherwise, add the `--region` option to each command\. For more information, see [Installing or updating the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html) and [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.

## Step 1: Create a VPC and subnets<a name="vpc-subnets-commands-example-create-vpc"></a>

The first step is to create a VPC and two subnets\. This example uses the CIDR block `10.0.0.0/16` for the VPC, but you can choose a different CIDR block\. For more information, see [VPC CIDR blocks](configure-your-vpc.md#vpc-cidr-blocks)\.

**To create a VPC and subnets using the AWS CLI**

1. Create a VPC with a `10.0.0.0/16` CIDR block using the following [create\-vpc](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc.html) command\.

   ```
   aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text
   ```

   The command returns the ID of the new VPC\. The following is an example\.

   ```
   vpc-2f09a348
   ```

1. Using the VPC ID from the previous step, create a subnet with a `10.0.1.0/24` CIDR block using the following [create\-subnet](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-subnet.html) command\.

   ```
   aws ec2 create-subnet --vpc-id vpc-2f09a348 --cidr-block 10.0.1.0/24
   ```

1. Create a second subnet in your VPC with a `10.0.0.0/24` CIDR block\.

   ```
   aws ec2 create-subnet --vpc-id vpc-2f09a348 --cidr-block 10.0.0.0/24
   ```

## Step 2: Make your subnet public<a name="vpc-subnets-commands-example-public-subnet"></a>

After you've created the VPC and subnets, you can make one of the subnets a public subnet by attaching an internet gateway to your VPC, creating a custom route table, and configuring routing for the subnet to the internet gateway\.

**To make your subnet a public subnet**

1. Create an internet gateway using the following [create\-internet\-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-internet-gateway.html) command\.

   ```
   aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
   ```

   The command returns the ID of the new internet gateway\. The following is an example\.

   ```
   igw-1ff7a07b
   ```

1. Using the ID from the previous step, attach the internet gateway to your VPC using the following [attach\-internet\-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-internet-gateway.html) command\.

   ```
   aws ec2 attach-internet-gateway --vpc-id vpc-2f09a348 --internet-gateway-id igw-1ff7a07b
   ```

1. Create a custom route table for your VPC using the following [create\-route\-table](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route-table.html) command\.

   ```
   aws ec2 create-route-table --vpc-id vpc-2f09a348 --query RouteTable.RouteTableId --output text
   ```

   The command returns the ID of the new route table\. The following is an example\.

   ```
   rtb-c1c8faa6
   ```

1. Create a route in the route table that points all traffic \(`0.0.0.0/0`\) to the internet gateway using the following [create\-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html) command\.

   ```
   aws ec2 create-route --route-table-id rtb-c1c8faa6 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-1ff7a07b
   ```

1. \(Optional\) To confirm that your route has been created and is active, you can describe the route table using the following [describe\-route\-tables](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-route-tables.html) command\.

   ```
   aws ec2 describe-route-tables --route-table-id rtb-c1c8faa6
   ```

   ```
   {
       "RouteTables": [
           {
               "Associations": [], 
               "RouteTableId": "rtb-c1c8faa6", 
               "VpcId": "vpc-2f09a348", 
               "PropagatingVgws": [], 
               "Tags": [], 
               "Routes": [
                   {
                       "GatewayId": "local", 
                       "DestinationCidrBlock": "10.0.0.0/16", 
                       "State": "active", 
                       "Origin": "CreateRouteTable"
                   }, 
                   {
                       "GatewayId": "igw-1ff7a07b", 
                       "DestinationCidrBlock": "0.0.0.0/0", 
                       "State": "active", 
                       "Origin": "CreateRoute"
                   }
               ]
           }
       ]
   }
   ```

1. The route table is currently not associated with any subnet\. You need to associate it with a subnet in your VPC so that traffic from that subnet is routed to the internet gateway\. Use the following [describe\-subnets](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-subnets.html) command to get the subnet IDs\. The `--filter` option restricts the subnets to your new VPC only, and the `--query` option returns only the subnet IDs and their CIDR blocks\.

   ```
   aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-2f09a348" --query "Subnets[*].{ID:SubnetId,CIDR:CidrBlock}"
   ```

   ```
   [
       {
           "CIDR": "10.0.1.0/24", 
           "ID": "subnet-b46032ec"
       }, 
       {
           "CIDR": "10.0.0.0/24", 
           "ID": "subnet-a46032fc"
       }
   ]
   ```

1. You can choose which subnet to associate with the custom route table, for example, `subnet-b46032ec`, and associate it using the [associate\-route\-table](https://docs.aws.amazon.com/cli/latest/reference/ec2/associate-route-table.html) command\. This subnet is your public subnet\.

   ```
   aws ec2 associate-route-table  --subnet-id subnet-b46032ec --route-table-id rtb-c1c8faa6
   ```

1. \(Optional\) You can modify the public IP addressing behavior of your subnet so that an instance launched into the subnet automatically receives a public IP address using the following [modify\-subnet\-attribute](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-subnet-attribute.html) command\. Otherwise, associate an Elastic IP address with your instance after launch so that the instance is reachable from the internet\.

   ```
   aws ec2 modify-subnet-attribute --subnet-id subnet-b46032ec --map-public-ip-on-launch
   ```

## Step 3: Launch an instance into your subnet<a name="vpc-subnets-commands-example-launch-instance"></a>

To test that your subnet is public and that instances in the subnet are accessible over the internet, launch an instance into your public subnet and connect to it\. First, you must create a security group to associate with your instance, and a key pair with which you'll connect to your instance\. For more information about security groups, see [Control traffic to resources using security groups](VPC_SecurityGroups.md)\. For more information about key pairs, see [Amazon EC2 Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) in the *Amazon EC2 User Guide for Linux Instances*\.

**To launch and connect to an instance in your public subnet**

1. Create a key pair and use the `--query` option and the `--output` text option to pipe your private key directly into a file with the `.pem` extension\. 

   ```
   aws ec2 create-key-pair --key-name MyKeyPair --query "KeyMaterial" --output text > MyKeyPair.pem
   ```

   In this example, you launch an Amazon Linux instance\. If you use an SSH client on a Linux or Mac OS X operating system to connect to your instance, use the following command to set the permissions of your private key file so that only you can read it\.

   ```
   chmod 400 MyKeyPair.pem
   ```

1. Create a security group in your VPC using the [create\-security\-group](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html) command\.

   ```
   aws ec2 create-security-group --group-name SSHAccess --description "Security group for SSH access" --vpc-id vpc-2f09a348
   ```

   ```
   {
       "GroupId": "sg-e1fb8c9a"
   }
   ```

   Add a rule that allows SSH access from anywhere using the [authorize\-security\-group\-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html) command\.

   ```
   aws ec2 authorize-security-group-ingress --group-id sg-e1fb8c9a --protocol tcp --port 22 --cidr 0.0.0.0/0
   ```
**Note**  
If you use `0.0.0.0/0`, you enable all IPv4 addresses to access your instance using SSH\. This is acceptable for this short exercise, but in production, authorize only a specific IP address or range of addresses\.

1. Launch an instance into your public subnet, using the security group and key pair you've created\. In the output, take note of the instance ID for your instance\.

   ```
   aws ec2 run-instances --image-id ami-a4827dc9 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-e1fb8c9a --subnet-id subnet-b46032ec
   ```
**Note**  
In this example, the AMI is an Amazon Linux AMI in the US East \(N\. Virginia\) Region\. If you're in a different Region, you'll need the AMI ID for a suitable AMI in your Region\. For more information, see [Finding a Linux AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html) in the *Amazon EC2 User Guide for Linux Instances*\.

1. Your instance must be in the `running` state in order to connect to it\. Use the following command to describe the state and IP address of your instance\.

   ```
   aws ec2 describe-instances --instance-id i-0146854b7443af453 --query "Reservations[*].Instances[*].{State:State.Name,Address:PublicIpAddress}"
   ```

   The following is example output\.

   ```
   [
       [
           {
               "State": "running",
               "Address": "52.87.168.235" 
           }
       ]
   ]
   ```

1. When your instance is in the running state, you can connect to it using an SSH client on a Linux or Mac OS X computer by using the following command:

   ```
   ssh -i "MyKeyPair.pem" ec2-user@52.87.168.235
   ```

   If you're connecting from a Windows computer, use the following instructions: [Connecting to your Linux instance from Windows using PuTTY](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html)\.

## Step 4: Clean up<a name="vpc-subnets-commands-example-clean-up"></a>

After you've verified that you can connect to your instance, you can terminate it if you no longer need it\. To do this, use the [terminate\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html) command\. To delete the other resources you've created in this example, use the following commands in their listed order:

1. Delete your security group:

   ```
   aws ec2 delete-security-group --group-id sg-e1fb8c9a
   ```

1. Delete your subnets:

   ```
   aws ec2 delete-subnet --subnet-id subnet-b46032ec
   ```

   ```
   aws ec2 delete-subnet --subnet-id subnet-a46032fc
   ```

1. Delete your custom route table:

   ```
   aws ec2 delete-route-table --route-table-id rtb-c1c8faa6
   ```

1. Detach your internet gateway from your VPC:

   ```
   aws ec2 detach-internet-gateway --internet-gateway-id igw-1ff7a07b --vpc-id vpc-2f09a348
   ```

1. Delete your internet gateway:

   ```
   aws ec2 delete-internet-gateway --internet-gateway-id igw-1ff7a07b
   ```

1. Delete your VPC:

   ```
   aws ec2 delete-vpc --vpc-id vpc-2f09a348
   ```