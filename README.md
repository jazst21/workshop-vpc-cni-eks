**EKS VPC CNI Workshop**

***

This is a custom workshop for EKS VPC CNI
**Getting started & Prerequisites**
**1\. Connecting to the AWS Workshop Studio**
To help you get hands-on as quickly as possible the AWS Container Immersion Day team has pre-created you AWS environment. You will need the participant hash, which should have been provided upon entry, and your email address to track your unique session.
The facilitator will provide you with an AWS account to deploy the components covered in this workshop. To access this AWS account you need to login to Workshop Studio.

* [Click here to open Workshop Studio](https://catalog.us-east-1.prod.workshops.aws/join) login page.
* Select **Email One-Time Password (OTP)** when prompted to sign-in.
* ![Application logo](/image/image-2.png)
* Enter your email address and select Send passcode
* ![Application logo](/image/image-3.png)
* Check your email for the one-time 9 digit passcode, enter it on the Workshop Studio page and select Sign in.
* ![Application logo](/image/image-4.png)
* Optional: if prompted for the Event access code, enter the access code shared with you and select Next
* ![Application logo](/image/image-5.png)
* You will be presented with terms and event details page. Please read and understand the terms governing use of Workshop Studio accounts. Click the checkbox to agree with the Terms and Conditions and Select **Join event** to continue.
* ![Application logo](/image/image-6.png)
* After you sign in to Workshop Studio, select **Open AWS Console** to access the AWS account provided by Workshop Studio Event. You can also copy the credentials to your own terminal.
* ![Application logo](/image/image-7.png)

**2\. Open Cloud 9 Environment**
Once you have logged into the AWS Management Console from your Workshop Studio, you will already have an EKS cluster and Cloud9 environment. Your Cloud 9 workspace will also have all the required tools installed in it.

* Navigate to [Cloud 9 ](https://console.aws.amazon.com/cloud9) in AWS Console.
* Click on **Open IDE** to open your Cloud 9 workspace.
* ![Application logo](/image/image-8.png)
* Close the welcome screen on Cloud 9 and wait for the terminal to be initialized.
* ![Application logo](/image/image-9.png)


**3\. Install additional tools**

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
    7. ![Application logo](/image/image-1.png)

**Lab-2 EKS VPC CNI with Terraform and EKS Blueprint**

1. make sure you have done "clone git repo" on previous Lab.
2. create new EKS cluster
3. 
4. deploy the application
5. 

**Lab-3 IP addressing Mode Use Case**

**Lab-4 NAT Use Case**

**All VPC CNI Use Cases:**
all use cases are described in referece : [https://docs.aws.amazon.com/eks/latest/userguide/pod-networking-use-cases.html](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking-use-cases.html)