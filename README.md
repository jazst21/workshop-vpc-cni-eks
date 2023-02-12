**EKS VPC CNI Workshop**

***

This is a custom workshop for EKS VPC CNI
**Getting started & Prerequisites**

* Log into your workshop
    1. go to xxxx.xxxx. select a slot for team and enroll into that account
    2. Copy paste user and password
    3. in your browser, create new incognito browser window freshly. use this window to run the workshop. do not use normal window otherwise you will risk using your actual (non workshop) aws account credentials & consumption.
    4. Log into your Cloud9 Editor and connect to the CLI
        * Install Terraform for linux on the cloud9 environment
        * `sudo apt-get update && sudo apt-get install -y gnupg software-properties-common`
        * reference: [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
    5. Install eksctl
        * `curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp`
        * `sudo mv /tmp/eksctl /usr/local/bin`
        * `eksctl version`
        * reference : [https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
        * 