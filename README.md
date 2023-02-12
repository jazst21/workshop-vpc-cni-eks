# EKS VPC CNI Workshop

This is a custom workshop for EKS VPC CNI

## **Getting started & Prerequisites**

### **1\. Connecting to the AWS Workshop Studio**

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

### **2\. Open Cloud 9 Environment**

Once you have logged into the AWS Management Console from your Workshop Studio, you will already have an EKS cluster and Cloud9 environment. Your Cloud 9 workspace will also have all the required tools installed in it.

* Navigate to [Cloud 9 ](https://console.aws.amazon.com/cloud9) in AWS Console.
* Click on **Open IDE** to open your Cloud 9 workspace.
* ![Application logo](/image/image-8.png)
* Close the welcome screen on Cloud 9 and wait for the terminal to be initialized.
* ![Application logo](/image/image-9.png)

### **3\. Install additional tools**

1. Install Terraform for linux on the cloud9 environment
    * `sudo apt-get update && sudo apt-get install -y gnupg software-properties-common`
    * `touch ~/.bashrc`
    * `terraform -install-autocomplete`
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

## **LAB-1 New EKS VPC CNI with eksctl**

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

## **Lab-2 EKS VPC CNI with Terraform and EKS Blueprint**

EKS Cluster w/ VPC-CNI Custom Networking
This example shows how to provision an EKS cluster with:

* AWS VPC-CNI custom networking to assign IPs to pods from subnets outside of those used by the nodes
* AWS VPC-CNI prefix delegation to allow higher pod densities - this is useful since the custom networking removes one ENI from use for pod IP assignment which lowers the number of pods that can be assigned to the node. Enabling prefix delegation allows for prefixes to be assigned to the ENIs to ensure the node resources can be fully utilized through higher pod densities. See the user data section below for managing the max pods assigned to the node.
* Dedicated /28 subnets for the EKS cluster control plane. Making changes to the subnets used by the control plane is a destructive operation - it is recommended to use dedicated subnets for the control plane that are separate from the data plane to allow for future growth through the addition of subnets without disruption to the cluster.

To disable prefix delegation from this example remove the environment environment variables `ENABLE_PREFIX_DELEGATION=true` and `WARM_PREFIX_TARGET=1` assignment from the `vpc-cni` addon

VPC CNI Configuration

In this example, the `vpc-cni` addon is configured outside of the `terraform-aws-eks` module even though the module supports configuring the `vpc-cni` addon. This is done to ensure the `vpc-cni` is updated *before* any EC2 instances are created so that the desired settings have applied before they will be referenced. In the `terraform-aws-eks` module, the addon resource has an explicit `depends_on` set to ensure the EKS managed node group(s), self-managed node group(s), and/or Fargate profile(s) are created *before* the addons. This is because nearly all of the addons require compute in order for them to reach a healthy state, otherwise they will fail and cause the `terraform apply` to fail. However, for the `vpc-cni` which is a daemonset, it does not require compute to exist first, and more importantly, it should be configured *before* compute is created.

With this configuration, you will now see that nodes created will have `--max-pods 110` configured do to the use of prefix delegation being enabled on the `vpc-cni`.

If you find that your nodes are not being created with the correct number of max pods (i.e. - for `m5.large`, if you are seeing a max pods of 29 instead of 110), most likely the `vpc-cni` was not configured *before* the EC2 instances.

Reference Documentation:

* [Documentation](https://docs.aws.amazon.com/eks/latest/userguide/cni-custom-network.html)
* [Best Practices Guide](https://aws.github.io/aws-eks-best-practices/reliability/docs/networkmanagement/#cni-custom-networking)

Prerequisites:

Ensure that you have the following tools installed locally:

1. [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
2. [kubectl](https://Kubernetes.io/docs/tasks/tools/)
3. [terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)

Deploy

To provision this example:

```
terraform init
terraform apply
```

Enter `yes` at command prompt to apply

Validate

The following command will update the `kubeconfig` on your local machine and allow you to interact with your EKS Cluster using `kubectl` to validate the deployment.

1. Run `update-kubeconfig` command:

```sh
aws eks --region <REGION> update-kubeconfig --name <CLUSTER_NAME>
```

2. List the nodes running currently

```sh
kubectl get nodes

# Output should look similar to below
NAME                                       STATUS   ROLES    AGE   VERSION
ip-10-0-34-74.us-west-2.compute.internal   Ready    <none>   86s   v1.22.9-eks-810597c
```

3. Inspect the nodes settings and check for the max allocatable pods - should be 110 in this scenario with m5.xlarge:

```sh
kubectl describe node ip-10-0-34-74.us-west-2.compute.internal

# Output should look similar to below (truncated for brevity)
  Capacity:
    attachable-volumes-aws-ebs:  25
    cpu:                         4
    ephemeral-storage:           104845292Ki
    hugepages-1Gi:               0
    hugepages-2Mi:               0
    memory:                      15919124Ki
    pods:                        110 # <- this should be 110 and not 58
  Allocatable:
    attachable-volumes-aws-ebs:  25
    cpu:                         3920m
    ephemeral-storage:           95551679124
    hugepages-1Gi:               0
    hugepages-2Mi:               0
    memory:                      14902292Ki
    pods:                        110 # <- this should be 110 and not 58
```

4. List out the pods running currently:

```sh
kubectl get pods -A -o wide

# Output should look similar to below
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE                                       NOMINATED NODE   READINESS GATES
kube-system   aws-node-ttg4h             1/1     Running   0          52s   10.0.34.74    ip-10-0-34-74.us-west-2.compute.internal   <none>           <none>
kube-system   coredns-657694c6f4-8s5k6   1/1     Running   0          2m    10.99.135.1   ip-10-0-34-74.us-west-2.compute.internal   <none>           <none>
kube-system   coredns-657694c6f4-ntzcp   1/1     Running   0          2m    10.99.135.0   ip-10-0-34-74.us-west-2.compute.internal   <none>           <none>
kube-system   kube-proxy-wnzjd           1/1     Running   0          53s   10.0.34.74    ip-10-0-34-74.us-west-2.compute.internal   <none>           <none>
```

5. Inspect one of the `aws-node-*` (AWS VPC CNI) pods to ensure prefix delegation is enabled and warm prefix target is 1:

```sh
kubectl describe pod aws-node-ttg4h -n kube-system

# Output should look similar below (truncated for brevity)
  Environment:
    ADDITIONAL_ENI_TAGS:                    {}
    AWS_VPC_CNI_NODE_PORT_SUPPORT:          true
    AWS_VPC_ENI_MTU:                        9001
    AWS_VPC_K8S_CNI_CONFIGURE_RPFILTER:     false
    AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG:     true # <- this should be set to true
    AWS_VPC_K8S_CNI_EXTERNALSNAT:           false
    AWS_VPC_K8S_CNI_LOGLEVEL:               DEBUG
    AWS_VPC_K8S_CNI_LOG_FILE:               /host/var/log/aws-routed-eni/ipamd.log
    AWS_VPC_K8S_CNI_RANDOMIZESNAT:          prng
    AWS_VPC_K8S_CNI_VETHPREFIX:             eni
    AWS_VPC_K8S_PLUGIN_LOG_FILE:            /var/log/aws-routed-eni/plugin.log
    AWS_VPC_K8S_PLUGIN_LOG_LEVEL:           DEBUG
    DISABLE_INTROSPECTION:                  false
    DISABLE_METRICS:                        false
    DISABLE_NETWORK_RESOURCE_PROVISIONING:  false
    ENABLE_IPv4:                            true
    ENABLE_IPv6:                            false
    ENABLE_POD_ENI:                         false
    ENABLE_PREFIX_DELEGATION:               true # <- this should be set to true
    MY_NODE_NAME:                            (v1:spec.nodeName)
    WARM_ENI_TARGET:                        1 # <- this should be set to 1
    WARM_PREFIX_TARGET:                     1
    ...
```

### **Lab-3 IP addressing Mode Use Case**

4. deploy the application
    1. `cd ~/environment/eks-app-mesh-polyglot-demo`
    2. `helm install workshop ~/environment/eks-app-mesh-polyglot-demo/workshop/helm-chart/`
    3. `kubectl get pod,svc -n workshop -o wide`
    4. `export LB_NAME=$(kubectl get svc frontend -n workshop -o jsonpath="{.status.loadBalancer.ingress[*].hostname}")`
    5. `echo $LB_NAME`
    6. go to browser paste & go. should be look like this
    7. ![Application logo](/image/image-1.png)
    8. 

### **Lab-4 NAT Use Case**

Destroy

To teardown and remove the resources created in this example:

```sh
terraform destroy -target=kubectl_manifest.eni_config -target=module.eks_blueprints_kubernetes_addons -auto-approve
terraform destroy -target=module.eks -auto-approve
terraform destroy -auto-approve
```

## **All VPC CNI Use Cases:**

all use cases are described in referece : [https://docs.aws.amazon.com/eks/latest/userguide/pod-networking-use-cases.html](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking-use-cases.html)