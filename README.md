# LSF-on-aws-cloud
lsf deployment on AWS

1.run 01-network.yaml in cloudformation

2.create EFS and remember DNS

3.run 022-lsf-master-global.yaml in cloudformation, fill DNS at EFSDns

4.run 031-dcv-login-server-global.yaml in cloudformation

current status
![image](https://user-images.githubusercontent.com/40814113/233983096-65178ca9-00df-4b5c-a750-33ec8226ced9.png)
