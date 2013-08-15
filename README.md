## CloudFormation Templates for SecureCloud Agent ##

Here are AWS CloudFormation templates for TrendMicro SecureCloud agents. You can use these templates in [AWS CloudFormation](http://aws.amazon.com/cloudformation/) to automatically launch an ebs data disk encrypted instance on AWS. The disk encryption key is managed by TrendMicro SecureCloud **K**ey **M**anagement **S**erver.

### Prerequisites ###

- SecureCloud [KMS](https://console.securecloud.com) login account with valid seats. (1 seat for 1 encryption disk)
- An AWS EC2 account that CloudFormation service and EC2 service are enabled.

### Usage ###

1. Download the latest template file archive from [GitHub](https://github.com/securecloud/CloudFormation/archive/master.zip)
2. Login to [AWS CloudFormation Console](https://console.aws.amazon.com/cloudformation/?region=ap-southeast-1) (you can switch to different region you like, to launch instances.)
3. Click "**Create New Stack**" and input your desired stack name, for example: "my-securecloud-test-instance"
4. Click "**Upload a Template File**", "**Browse**", choose a template file(platform) you have downloaded at Step 1. and click "Continue"
5. Input "**KeyPairName**", the EC2 KeyPairs used to access EC2 instances.
6. Input "**PASSPHRASE**", [SecureCloud KMS Console](https://console.securecloud.com) -> Administration -> User Management -> Provision passphrase
7. Input "**ACCOUNTID**", [SecureCloud KMS Console](https://console.securecloud.com) -> Administration -> User Management -> Account ID
8. Input "**InstanceType**", allowed types are "m1.small", "m1.medium", "m1.large", "m1.xlarge", "m2.xlarge", "m2.2xlarge", "m2.4xlarge", "c1.medium", "c1.xlarge". 
9. Click "Continue".
10. (Optional) Input a **key/value pair** for instance tag, for example: "Name/my-securecloud-test-instance-01" 
11. Click "Continue".
12. Review your final settings for CloudFormation stack creation.
13. Click "Continue" to lauch stack
14. In CloudFomration Stacks management console, wait for stack creation to complet (about 5~10 mins).
15. After creation complete, in [SecureCloud KMS Console](https://console.securecloud.com), you can see the updated ebs data disk information and the progress of encryption.

### FAQ ###
- Why need to input KeyPairName?

	This is the key-pair assigned to the newly instance that you lauched.details please refer [Amazon EC2 Key Pairs doc](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

- How can I change the ebs volume size or add more ebs volumes into the instance that can protected by SecureCloud?
	
	You can modify the templeate to add more resources when creating CloudFormation stacks. You can add more deivce name and change the size. please noted, the DeviceName can't be duplicated.

 
        "BlockDeviceMappings" : [
          {"DeviceName" : "/dev/sdm",
            "Ebs" : { "VolumeSize" : "1" }
          },
          {"DeviceName" : "/dev/sdn",
            "Ebs" : { "VolumeSize" : "2" }
          },
		  {"DeviceName" : "/dev/sdo",
			"Ebs" : { "VolumeSize" : "size_you_want"}
		  }],
  

	If add more ebs volumes, we also need to modify "UserData" section for new disk partition, create filesystem and mount point.

        "UserData" : { "Fn::Base64" : { "Fn::Join" : ["", [
          "#!/bin/bash\n",
          "(echo n;echo p;echo 1;echo;echo;echo w) | fdisk /dev/xvdm\n",          
          "(echo n;echo p;echo 1;echo;echo;echo w) | fdisk /dev/xvdn\n",
		  "(echo n;echo p;echo 1;echo;echo;echo w) | fdisk /dev/xvdo\n",
          "mkfs.ext3 /dev/xvdm1\n",
          "mkfs.ext3 /dev/xvdn1\n",
		  "mkfs.ext3 /dev/xvdo1\n",
          "mkdir -p /securedisk/disk1\n",
          "mkdir -p /securedisk/disk2\n",
		  "mkdir -p /securedisk/disk3\n",
          "mount /dev/xvdm1 /securedisk/disk1\n",
          "mount /dev/xvdn1 /securedisk/disk2\n",
		  "mount /dev/xvdn1 /securedisk/disk3\n",

