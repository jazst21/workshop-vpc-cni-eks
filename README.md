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
    4. 