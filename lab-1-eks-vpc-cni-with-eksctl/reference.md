https://eksctl.io/usage/addons/
https://blog.sivamuthukumar.com/eks-high-pod-density
https://docs.aws.amazon.com/eks/latest/userguide/cni-iam-role.html
https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/policies/arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy$jsonEditor
brew upgrade eksctl && { brew link --overwrite eksctl; } || { brew tap weaveworks/tap; brew install weaveworks/tap/eksctl; }
aws eks --region us-west-2 update-kubeconfig --name eks-vpc-cni-eksctl-1
aws eks describe-addon-versions --addon-name vpc-cni
bug issues & solution/workaround on top of existing documentation (eksctl/terraform):
https://github.com/weaveworks/eksctl/issues/5134
https://github.com/aws/amazon-vpc-cni-k8s/issues/1571
https://eksctl.io/announcements/nodegroup-override-announcement/
https://github.com/aws/containers-roadmap/issues/1333
https://aws.amazon.com/blogs/containers/amazon-eks-add-ons-advanced-configuration/
