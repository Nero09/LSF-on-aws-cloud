# LSF-on-aws-cloud
lsf deployment on AWS

architecture

<img width="784" alt="Screenshot 2023-04-27 at 12 35 08" src="https://user-images.githubusercontent.com/40814113/234760295-a3871584-af5c-48fc-bd26-657e9b146d0b.png">

1.run 01-network.yaml in cloudformation

2.create EFS in console, pay attention 4 parts: a.customize(自定义) when create;b.choose subnet(xxx-network-yourcluster);c.choose sg(yourcluster-private-subnet);d remember DNS

3.run 022-lsf-master-global.yaml in cloudformation, fill DNS at EFSDns. Then in $FileSystemMountPoint/$LSFInstallPath/$YourCluster/10.1/resource_connector/aws/scripts/user_data.sh, change (mount -t nfs -o rw,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 $FSXN_SVM_DNS_NAME:/vol1) to (mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport $FSXN_SVM_DNS_NAME:/ $NFS_MOUNT_POINT)

4.in EC2 console, change LSFMasterEC2 iam role to a role with administator access (just for test, I will modify this item with a manual created iam role later)

5.run 031-dcv-login-server-global.yaml in cloudformation

current status
![image](https://user-images.githubusercontent.com/40814113/233983096-65178ca9-00df-4b5c-a750-33ec8226ced9.png)



