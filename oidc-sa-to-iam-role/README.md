This method associates the open id for a EKS cluster with an Service Account in K8S

When you create an EKS cluster, it creates an OIDC end point

Step 1: create an IAM OIDC provider which links to cluster OIDC

    eksctl utils associate-iam-oidc-provider \
    --cluster eks-demo \
    --region us-west-2 \
    --approve

Step2: Associate a SA to an IAM role, all in one command
Creates SA
Creates IAM role

    eksctl create iamserviceaccount \
    --name sa-poc
    --cluster eks-demo
    --attach-policy-arn <policy-ARN>
    --approve
    --region us-west-2
    --namespace default

Step3: Create a POD with the service account - sa-poc. Use image since it has awscli in it

    --image: sdscello/awscli:latest

Step4: After you create the POD with the above image and SA, 

    aws sts get-caller-identity




