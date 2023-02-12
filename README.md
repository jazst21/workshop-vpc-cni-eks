**EKS VPC CNI Workshop**

***

This is a custom workshop for EKS VPC CNI
**Getting started & Prerequisites**

1. Login to AWS Workshop Portal
    This workshop creates an AWS account and a Cloud9 environment. You will need the Participant Hash provided upon entry, and your email address to track your unique session.
    Connect to the portal by clicking the button or browsing to https://dashboard.eventengine.run/
    . The following screen shows up.
2. Copy paste user and password
3. in your browser, create new incognito browser window freshly. use this window to run the workshop. do not use normal window otherwise you will risk using your actual (non workshop) aws account credentials & consumption.
4. Log into your Cloud9 Editor and connect to the CLI
    1. Install Terraform for linux on the cloud9 environment
        * `sudo apt-get update && sudo apt-get install -y gnupg software-properties-common`
        * reference: [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
    2. Install eksctl
        * `curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp`
        * `sudo mv /tmp/eksctl /usr/local/bin`
        * `eksctl version`
        * reference : [https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
    3. Install kubectl
        * `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`
        * `curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"`
        * `echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check`
        * `sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`
        * `kubectl version --client`

**LAB-1 New EKS VPC CNI with eksctl**

1. Clone git repo for this workshop
    * `git clone https://github.com/jazst21/workshop-vpc-cni-eks`
    * `cd workshop-vpc-cni-eks`
2. create new EKS cluster
    1. create using eksctl
    2. change k8s context
3. deploy the application
    1. `cd ~/environment/eks-app-mesh-polyglot-demo`
    2. `helm install workshop ~/environment/eks-app-mesh-polyglot-demo/workshop/helm-chart/`
    3. `kubectl get pod,svc -n workshop -o wide`
    4. `export LB_NAME=$(kubectl get svc frontend -n workshop -o jsonpath="{.status.loadBalancer.ingress[*].hostname}")`
    5. `echo $LB_NAME`
    6. go to browser paste & go. should be look like this
    7. ![Application logo](/image/image-1.png "application")

**Lab-2 EKS VPC CNI with Terraform and EKS Blueprint**

1. make sure you have done "clone git repo" on previous Lab.
2. create new EKS cluster
3. 
4. deploy the application
5. 

**Lab-3 IP addressing Mode Use Case**

**Lab-4 NAT Use Case**

<br>
