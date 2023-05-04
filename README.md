# LSF-on-aws-cloud
lsf deployment on AWS

## Architecture

<img width="784" alt="Screenshot 2023-04-27 at 12 35 08" src="https://user-images.githubusercontent.com/40814113/234760295-a3871584-af5c-48fc-bd26-657e9b146d0b.png">

## Prerequisites:
1.An AWS account with administrative level access  
2.Software and Licenses for IBM Spectrum LSF 10.1
|Kind|IBM Download Source|Description|Package Name|
|:---|:-----|:----------|------------|
|Install script|Passport Advantage|--|`lsf10.1_lsfinstall_linux_x86_64.tar.Z` 
|Base distribution|Passport Advantage|--|`lsf10.1_linux2.6-glibc2.3-x86_64.tar.Z`
|Entitlement file|Passport Advantage|--|`lsf_std_entitlement.dat` or `lsf_adv_entitlement.dat`
|Fix Pack 10|Fix Central|lsf-10.1.0.10-spk build545500|`lsf10.1_linux2.6-glibc2.3-x86_64-545500.tar.Z`|
|Patch|Fix Central|LSF Resource Connector patch to address Log4J CVE-2021-44228 security vulnerability.|`lsf10.1_linux2.6-glibc2.3-x86_64-600877.tar.Z`|  

3.An Amazon EC2 key pair  
4.A free subscription to the AWS FPGA Developer AMI from AWS Marketplace

## Deployment:
1. Run 01-network.yaml via cloudformation

2. Create EFS in console, pay attention to 4 parts: 
  * customize(自定义) when create;
  * choose subnet(xxx-network-yourcluster);
  * choose security group(yourcluster-private-subnet);
  * remember DNS

3. Run 022-lsf-master-global.yaml via cloudformation
  LSF software path are the S3 buckets where you locate your LSF software & licence. EFSDns is DNS in step 2.  
  Then in file `$FileSystemMountPoint/$LSFInstallPath/$YourCluster/10.1/resource_connector/aws/scripts/user_data.sh`,  
  replace  
  `mount -t nfs -o rw,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 $FSXN_SVM_DNS_NAME:/vol1`  
  to  
  `mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport $FSXN_SVM_DNS_NAME:/ $NFS_MOUNT_POINT`

4. At EC2 console, change LSFMasterEC2 iam role to a role with administator access (just for test, I will modify this item with a manual created iam role later)

5. Run 031-dcv-login-server-global.yaml via cloudformation

In the Stack Failure Options, we recommend choosing **Preserve successfully provisioned resources**. This preserves the resources of the CloudFormation Stack instead of cleaning up stack on deployment failure, thereby facilitating debug. 
