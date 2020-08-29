Launch cloud9

Ensure your role is the one that you created 

    arn:aws:sts::854394523447:assumed-role/eks-admin-role-for-cloud9/i-01dea760482e492f1

Launch your cluster now

Check all the cluster roles and cluster role bindings create

    k get clusterrole
    k get clusterrolebinding

=====Instructions to map IAM USER to a K8S user====

Review the clusterrolebinding called "view" since this is used in the user_binding.yaml

    kcf user_binding.yaml
    
In the AWS console, create an IAM user - eksdev or eksview in IAM, get the IAM user ARN
Get the SA/SAK so you can test it

Back in Cloud9 (after you have created your cluster)

    eksctl get iamidentitymapping --cluster eks-demo --region us-west-1

will show mapping from cm, this will update the cm, aws-auth

    eksctl create iamidentitymapping --cluster eks-demo --region us-west-1 --arn " " --username dev

    eksctl get iamidentitymapping --cluster eks-demo --region us-west-1

    kubectl get cm $KN -o yaml

To test,
create a new profile use aws configure

    aws configure --profile eksdev

    aws eks update-kubeconfig --name eks-demo --profile eksdev

    kgs

    kgp

    kgn

    kubectl create namespace dev


=====Instructions to map ROLE to a service account====

The next command will create serviceAccount: dev and bind to ClusterRole "view"

    kcf role_binding.yaml

The ServiceAccount name in the above file is called "dev"

Create a new IAM ROLE in AWS, call it, no policy required
    eks-dev-role

Get the ARN of the ROLE

Establish trust between eks-dev-role and user created from previous step - eksdev

For the user "eksdev" grant it AssumeRole policy via the "sts" service

Now edit the cm in kube-system namespace, use the ARN of the ROLE

    eksctl create iamidentitymapping --cluster eks-demo --region us-west-1 --arn " " --username system:serviceaccount:kube-system:dev --group system:serviceaccount:kube-system

    kubectl get cm $KN -o yaml    

To delete the previous user arn, use the following command (no need to delete)
    
    eksctl delete iamidentitymapping --cluster eks-demo --region us-west-1 --arn "userARN"

Regenerate kubeconfig

    aws eks update-kubeconfig --name eks-demo --role-arn "roleARN" --profile eksdev

    kgs

    kgp

    kgn

    kubectl create namespace dev





=====Instructions to install kube2iam and use it ======

Concept: Since a POD is like a node, it inherits the IAM role attached to the node.  That gives it privileged access which is not required for the POD.

Using kube2iam, you can control that by doing the following
- Create an IAM role for the POD (based on the AWS services it needs to access)
- At the namespace level (the pod namespace), create an annotation by referencing the role
- In the deployment definition, reference the same role via an annotation


    kcf kube2iam.yaml

    kgp -n utilities

    k logs <podname> -n utilities

Create a ns dev
Create a role with S3 access - eks-dev-ns-role

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

   annotation:
     iam.amazonaws.com/allowed-roles: |
      ["eks-dev-ns-role ARN"]

Similarly, in the deployment for nginx in namespace, dev
metadata:

   annotation:
     iam.amazonaws.com/role: ["eks-dev-ns-role ARN"]

    curl 169.254.169.254/latest/meta-data/iam/security-credentials








