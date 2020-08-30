ALB Ingress Controller
=======================

In the private subnet for the cluster, review the tags.  For the public subnet(s) add the tags.  To know the tag name, look a the private subnet with "shared" value

Create an IAM role for ALB Ingress Controller, will use a custom policy

    eks-alb-role

Copy and paste from alb_ingress_role_policy.json

Build trust relationship, copy the node role ARN and in the ALB role

    ,"AWS": "nodeRole ARN"

In the namespace, ingress-nginx, annotate eks-alb-role ARN 

Copy the eks-alb-role ARN and add annotation in the deployment, alb-ingress-controller.yaml, to refer eks-alb-role ARN 

    --aws-region=us-west-2

    kcf rbac-role.yaml

    kcf alb-ingress-controller.yaml

When the controller gets deployed, the LB is not created, but gets created when the ingress is created. As next step....


=====ALB Ingress resource========

FOR ALB, service can be NodePort or LoadBalancer

    kcf deployment/alb_service.yaml

    kgs

Now, deploy alb-ingress, check the annotations before you deploy

    kcf deployments/alb_ingress.yaml

In the EKS cluster, edit the additional SG to add the LB security group.  This is a nuisance, so create a security group - alb-ingress-sg with inbound rules http and https (0.0.0.0/0), and now associate the new SG with the EKS cluster additional SG (all traffic)

With this new SG created, now associate via annotation in the ALB ingress (alb_ingress.yaml)  There are other annotations that can be configured as well.

    alb.ingress.kubernetes.io/security-groups: <sgname>

    alb.ingress.kubernetes.io/listebn-ports: '[{HTTPS:443}, {HTTP:80}]'

In R53, a new record will be created for ELB, how is this configured is next



=====R53 External DNS configuration=======

Ingress creates LB - Route53 - points to this LB.  This section will discuss automatic configuration of R53-->LB-->ALB IC

Create an IAM role - eks-external-dns-role (route53 access)

Node - Trust relationship

eks-external-dns-role trusts node role

Annotate namespace - external-dns with the above role

Under Ingres --> files are 
    external-dns.yaml


