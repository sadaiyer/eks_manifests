Launch cloud9
Ensure your role is the one that you created 
arn:aws:sts::854394523447:assumed-role/eks-admin-role-for-cloud9/i-01dea760482e492f1

Launch your cluster now
Check all the cluster roles and cluster role bindings create
k get clusterrole
k get clusterrolebinding

Review the clusterrolebinding called "view" since this is used in the user_binding.yaml

Create a user - eksdev or eksview in IAM, get the IAM user ARN
Get the SA/SAK so you can test it

eksctl get iamidentitymapping --cluster eks-demo --region us-west-1
- will show mapping from cm

eksctl create imaidentitymapping --cluster eks-demo --region us-west-1 --arn " " --username dev

eksctl get iamidentitymapping --cluster eks-demo --region us-west-1

kubectl get cm $KN -o yaml

To test,
create a new profile use aws configure
aws configure --profile eksdev

aws eks update-kubeconfig --name eks-demo --profile eksdev

kgs
