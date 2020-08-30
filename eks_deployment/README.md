
Using EKSCTL:
Install eksctl - https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
Upgraded AWS CLI to v2
Created IAM ROLE with admin access 
Attached above role to cloud9 ec2 instance
arn:aws:sts::854394523447:assumed-role/eks-admin-role-for-cloud9/i-01dea760482e492f1
 
Confirmed in Cloud9 before and after attaching
aws sts get-caller-identity
Need to have a VPC with public and private subnets, with private subnet having access to internet via NAT Gateway 
 
eks.yaml
————
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: eks-demo
  region: us-west-2
  version: "1.16"
vpc:
  id: "vpc-8544a1e3"
  cidr: "10.0.0.0/16"
  subnets:
    private:
      us-west-2a:
          id: "subnet-0546080f496037415"
          cidr: "10.0.3.0/24"
      us-west-2b:
          id: "subnet-86e4adcf"
          cidr: "10.0.2.0/24"
 
managedNodeGroups:
  - name: eks-ng
    minSize: 2
    maxSize: 2
    desiredCapacity: 2
    instanceType: t3.small
    volumeSize: 5
    privateNetworking: true
    ssh:
      allow: true
      publicKeyPath: si-eks-kp
      #sourceSecurityGroupIds: ["sg-0e10bb986d85449b8"]
    labels: {env: dev}
    tags:
      costid: devops
    iam:
      withAddonPolicies:
        externalDNS: true
        autoScaler: true
        ebs: true
        efs: true
        cloudWatch: true
        albIngress: true
 
eksctl create cluster -f my_eksctl_ctl.yaml
 
eksctl scale nodegroup --cluster=eks-demo —name=eks-ng —nodes=4 —nodes-min=2 —nodes-max=2 —region=us-west-2
 
eksctl delete cluster -f my_eksctl_ctl.yaml
