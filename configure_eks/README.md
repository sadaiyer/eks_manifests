Launch cloud9

Ensure your role is the one that you created 

    arn:aws:sts::<aws-account-num>:assumed-role/eks-admin-role-for-cloud9/i-01dea760482e492f1

Launch your cluster now - https://github.com/sadaiyer/eks_manifests/blob/master/eks_deployment/eksctl_eks.yaml or uploading here too...(Mar 14, 2021)

    eksctl create cluster -f my_eksctl_eks.yaml

 To create the nodegroup:

    eksctl scale nodegroup --cluster=eks-demo —name=eks-ng —nodes=4 —nodes-min=2 —nodes-max=2 —region=us-west-2
 
 To delete the cluster (cleanup):

    eksctl delete cluster -f my_eksctl_eks.yaml




=====Instructions to map IAM USER to a K8S user=====

Concepts: In EKS, an IAM user (or role) gets mapped to K8S user or SA via "aws-auth" config map.  The next step of instructions will create a user in k8s as "dev" and map to IAM user called - "eksdev"

To check all the cluster roles and cluster role bindings, run, and specifically look for the "view" cluster role and clusterrolebinding

    k get clusterrole
    k get clusterrolebinding

Review the clusterrolebinding called "view" since this is used in the user_binding.yaml

    kcf user_binding.yaml
    
In the AWS console, create an IAM user - "eksdev" in IAM, provide "AmazonEKSWorkerNodePolicy" get the IAM user ARN
Get the SA/SAK so you can test it

Back in Cloud9 (after you have created your cluster)

To view configmap, "aws-auth"

    eksctl get iamidentitymapping --cluster eks-demo --region us-west-2

Now update the configmap, aws-auth, with the newly created username ARN

    eksctl create iamidentitymapping --cluster eks-demo --region us-west-2 --arn "<EnterARN>" --username dev

    eksctl get iamidentitymapping --cluster eks-demo --region us-west-2

Verify that the configmap got updated,

    kubectl get cm aws-auth $KN -o yaml

To test,
create a new profile use aws configure

    aws configure --profile eksdev

    aws eks update-kubeconfig --name eks-demo --profile eksdev --region us-west-2

    kgs

    kgp

    kgn

    kubectl create namespace dev

Creating namespace will error out since this is a "view" user



=====Instructions to map ROLE to a service account====

The next command will create serviceAccount: dev and bind to ClusterRole "view"

    kcf role_binding.yaml

The ServiceAccount name in the above file is called "dev"

Create a new IAM ROLE in AWS, call it, no policy required
    eks-dev-role

Get the ARN of the ROLE

Establish trust between eks-dev-role (EC2 service) and user created from previous step - eksdev

For the user "eksdev" grant it AssumeRole policy via the "sts" service

Now edit the cm in kube-system namespace, use the ARN of the ROLE

    eksctl create iamidentitymapping --cluster eks-demo --region us-west-2 --arn "<roleARN>" --username system:serviceaccount:kube-system:dev --group system:serviceaccount:kube-system

    kubectl get cm aws-auth $KN -o yaml    

To delete the previous user arn, use the following command (no need to delete!)
    
    eksctl delete iamidentitymapping --cluster eks-demo --region us-west-2 --arn "userARN"

Regenerate kubeconfig

    aws eks update-kubeconfig --name eks-demo --role-arn "roleARN" --profile eksdev --region us-west-2

    kgs

    kgp

    kgn

    kubectl create namespace dev

kgn above gives error, as well as create namespace

=====Instructions to install kube2iam and use it ======

Concepts: Since a POD is like a node, it inherits the IAM role attached to the node.  That gives it privileged access which is not required for the POD.

Using kube2iam, you can control that by doing the following
Create an IAM role for the POD (based on the AWS services it needs to access)

Annotate at 2 place, namespace level and deployment level:

At the namespace level (the pod namespace), create an annotation by referencing the role
In the deployment definition, reference the same role via an annotation

    kcf kube2iam.yaml

The kube2iam creates the necessary namespace, service account, cr and crb, along with a daemonset.

    kgp -n utilities

    k logs <podname> -n utilities

Create a ns dev

    k create namespace dev

Create a role (service: EC2) with S3 access - eks-dev-ns-role

In the node role, grant AssumeRole via "sts"

For the eks-dev-ns-role, create a trust with node role

Now in the deploy.yaml for say, nginx, in dev namespace

Before kube2iam, to test and verify the node attached to the POD

exec into the POD, and install the utilities

    apt-get update

    apt-get install awscli -y

    aws sts get-caller-identity

NodeRole is visible, to change

In the namespace dev, create an annotation
metadata:

   annotations:
     iam.amazonaws.com/allowed-roles: |
       ["eks-dev-ns-role ARN"]

    k get ns dev -o yaml

Similarly, in the deployment for nginx in namespace, dev.  This needs to be added at the POD level definition in the deployment.

metadata:

   annotations:
     iam.amazonaws.com/role: eks-dev-ns-role-ARN

    curl 169.254.169.254/latest/meta-data/iam/security-credentials








