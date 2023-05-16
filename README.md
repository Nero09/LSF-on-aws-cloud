# LSF-on-aws-cloud

## Architecture

![LSF diagram drawio](https://github.com/Nero09/LSF-on-aws-cloud/assets/40814113/b0536d88-045a-4e83-b455-55dfe30e0838)


## Prerequisites:
1.An AWS account at **Beijing or Ningxia region** with administrative level access  
2.Software and Licenses for IBM Spectrum LSF 10.1
|Kind|IBM Download Source|Description|Package Name|
|:---|:-----|:----------|------------|
|Install script|Passport Advantage|--|`lsf10.1_lsfinstall_linux_x86_64.tar.Z` 
|Base distribution|Passport Advantage|--|`lsf10.1_linux2.6-glibc2.3-x86_64.tar.Z`
|Entitlement file|Passport Advantage|--|`lsf_std_entitlement.dat` or `lsf_adv_entitlement.dat`
|Fix Pack 10|Fix Central|--|`lsf10.1_linux2.6-glibc2.3-x86_64-601088.tar.Z`|

3.An Amazon EC2 key pair  
4.A free subscription to the AWS FPGA Developer AMI from AWS Marketplace, detail process as below.  
如果想使用 AWS vivado AMI, 需要在 AWSMarkerplace 订阅该 AMI。进入 https://awsmarketplace.amazonaws.cn/marketplace/search/results?x=0&y=0&searchTerms=Fpga 后，搜索 FPGA 
![image](https://user-images.githubusercontent.com/40814113/236984177-65d7b513-1096-427a-ad00-e2a8e303244b.png)  
选择 【FPGA Developer AMI】点击订阅  
![image](https://user-images.githubusercontent.com/40814113/236984256-1520dc1b-e8cf-49ba-971b-3aeb7e1a5cc5.png)  
验证已经成功订阅，进入marketplace  
![image](https://user-images.githubusercontent.com/40814113/236984385-953c621e-ac12-497b-b1d4-1e0e5679536f.png)  
在管理订阅栏目中看到已经成功订阅FPGA AMI  
![image](https://user-images.githubusercontent.com/40814113/236984672-7e90478b-7f2d-40cd-b6aa-e4b8145579b5.png)


## You can select 1 of blew file systems to deploy
## A.Deployment with EFS file system:
1. Run 01-network.yaml via cloudformation, input stack name, choose 1 az, and keep other default. Wait status become "create_complete"  

2. Run 012-efs.yaml to create EFS shared file storage, input stack name and keep other default. Wait status become "create_complete"  

3. Run 021-lsf-master-cn.yaml via cloudformation, input stack name and choose your keypair, keep other default. Wait status become "create_complete"  
  LSF software path are the S3 buckets where you locate your LSF software & licence.   

4. Run 031-dcv-login-server-cn.yaml via cloudformation, input stack name and choose your keypair, keep other default.Wait status become "create_complete"  

In the Stack Failure Options, we recommend choosing **Preserve successfully provisioned resources**. This preserves the resources of the CloudFormation Stack instead of cleaning up stack on deployment failure, thereby facilitating debug. 

## B.Deployment with FSx for Lustre file system:
1. Run 011-network-lustre.yaml via cloudformation, input stack name, choose 1 az, and keep other default. Wait status become "create_complete"  

2. Run 013-fsxforlustre.yaml to create EFS shared file storage, input stack name and keep other default. Wait status become "create_complete"  

3. Run 023-lsf-master-cn-lustre.yaml via cloudformation, input stack name and choose your keypair, keep other default. Wait status become "create_complete"  
  LSF software path are the S3 buckets where you locate your LSF software & licence.   

4. Run 033-dcv-login-server-cn-lustre.yaml via cloudformation, input stack name and choose your keypair, keep other default.Wait status become "create_complete"  

In the Stack Failure Options, we recommend choosing **Preserve successfully provisioned resources**. This preserves the resources of the CloudFormation Stack instead of cleaning up stack on deployment failure, thereby facilitating debug. 

## Test:
1. Log into the login server via SSH as `centos` user using the private key from the key pair you provided in the Cloudformation stack and the IP address found in **LoginServerPublicIp** under the stack031's **Outputs** tab.

   `ssh -i /path/to/private_key centos@<host_ip>`

   >NOTE: If you have trouble connecting, ensure the security group on the login server includes the IP address of your client.

2. Run the `lsid` command to verify that LSF installed properly and is running. You should see something similar to the following:
    ```
    IBM Spectrum LSF Standard 10.1.0.11, Nov 12 2020
    Copyright International Business Machines Corp. 1992, 2016.
    US Government Users Restricted Rights - Use, duplication or disclosure restricted by GSA ADP Schedule Contract with IBM Corp.
    My cluster name is <CLUSTER_NAME>
    My master name is <MASTER_HOST_NAME>
    ```
3. Download and install the [NICE DCV remote desktop native client](https://download.nice-dcv.com) on your local laptop/desktop computer.  
  a. Launch the DCV client application.  
  b. Paste the public IP address of the **Login Server** into the field. Click "Trust & Connect" when prompted.  
  c. Enter the Username and Password.  You can find these credentials in **AWS Secrets Manager** in the AWS Console: Go to the Secrets Manager service and select the ***/DCVCredentialsSecret** secret, then click on the **Retrieve secret value** button. Copy the **username** and **password** and paste them into the appropriate DCV client fields.

4. In DCV linux console, **Submit the setup job into LSF**. The `--scratch-dir` should be the path to the scratch directory you defined when launching the CloudFormation stack in the previous tutorial.  The default is `/efs/scratch`.

   `bsub -R aws -J "setup" sleep 5 --scratch-dir /efs/scratch`
5. Watch job status. This job will generate demand to LSF Resource Connector for an EC2 instance.  Shortly after you submit the job, you should see a new "LSF Exec Host" instance in the EC2 Dashboard in the AWS console. It should take 2-5 minutes for this new instance to join the cluster and accept the job.  Use the `bjobs` command to watch the status of the job. 

### Clean up

To help prevent unwanted charges to your AWS account, you can delete the AWS resources that you used for this tutorial.
You can delete stack as sequence 031->021->012->01 or 033->023->013->011
