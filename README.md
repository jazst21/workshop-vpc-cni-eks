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
    1. create iam-role
        * We'll use default iam role for eks vpc cni role
    2. create using eksctl
        * `eksctl create cluster -f lab-1-eks-vpc-cni-with-eksctl/clusterconfig-1.yaml`
    3. change k8s context
        * `aws eks update-kubeconfig --region us-east-1 --name eks-vpc-cni-eksctl-1`
3. deploy the application
    1. `helm install workshop /eks-app-mesh-polyglot-demo/workshop/helm-chart/`
    2. `helm ls -n workshop`
    3. `kubectl get pod,svc -n workshop -o wide`
    4. `export LB_NAME=$(kubectl get svc frontend -n workshop -o jsonpath="{.status.loadBalancer.ingress[*].hostname}")`
    5. `echo $LB_NAME`
    6. go to browser paste & go. should be look like this
    7. ![Application logo](/image/image-1.png)

##### Calculate the number of maximum pods per node.

Download the max pods per node calculator script and change the access mode

```sh
curl -o max-pods-calculator.sh https://raw.githubusercontent.com/awslabs/amazon-eks-ami/master/files/max-pods-calculator.sh
chmod +x max-pods-calculator.sh
```

Calculate the max pods using the max pods calculator script. You can pass the instance type and CNI version to calculate the max pods per node.

```sh
./max-pods-calculator.sh \\
--instance-type \<m5.large> \\
--cni-version <1.9.x-eksbuild.y> \\
--cni-prefix-delegation-enabled
```

You can take a look at the max pods calculator script. Without enabling the prefix delegation feature or non-nitro instance types, the calculation of the pods are ENI \* (# of IPv4 per ENI - 1) + 2.
The calculation changed into ENI ((# of IPv4 per ENI - 1) 16) + 2 where 16 is IPS\_PER\_PREFIX. The pod density per node is significantly more than before.
Create one of the following node groups with at least one Amazon EC2 Nitro Amazon Linux 2 instance type. See the list of Nitro Amazon Linux 2 Instance Types.
Once your nodes are deployed, you can run the kubectl describe command in the node to see the max pods in the allocatable.

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
create new tab in cloud9. move to lab-2 terraform folder
`lab-2-eks-vpc-cni-with-terraform/vpc-cni-custom-networking`
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
aws eks --region us-east-1 update-kubeconfig --name eks-vpc-cni-terraform-1
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

SNAT for pods

If you deployed your cluster using the IPv6 family, then the information in this topic isn't applicable to your cluster, because IPv6 addresses are not network translated. For more information about using IPv6 with your cluster, see Tutorial: Assigning IPv6 addresses to pods and services.

By default, each pod in your cluster is assigned a private IPv4 address from a classless inter-domain routing (CIDR) block that is associated with the VPC that the pod is deployed in. Pods in the same VPC communicate with each other using these private IP addresses as end points. When a pod communicates to any IPv4 address that isn't within a CIDR block that's associated to your VPC, the Amazon VPC CNI plugin for Kubernetes

translates the pod's IPv4 address to the primary private IPv4 address of the primary elastic network interface of the node that the pod is running on, by default \*.

###### Due to this behavior:

1. Resources that are in networks or VPCs that are connected to your cluster VPC using VPC peering, a transit VPC, or Direct Connect can't initiate communication to your pods. Your pods can initiate communication to those resources and receive responses from them though.
2. Your pods can communicate with internet resources only if the node that they're running on has a public or elastic IP address assigned to it and is in a public subnet. A public subnet's associated route table has a route to an internet gateway. We recommend deploying nodes to private subnets, whenever possible.

If you have resources in networks or VPCs that are connected to your cluster VPC using VPC peering, a transit VPC, or Direct Connect that need to initiate communication with your pods using an IPv4 address, then you need to change the default configuration with the following command.

```sh
kubectl set env daemonset -n kube-system aws-node AWS_VPC_K8S_CNI_EXTERNALSNAT=true
```

If you've changed the setting to true and want your pods to communicate to the internet, then the route table that's associated with the private subnet that your node is deployed in must contain a route to a public NAT gateway.

\*If a pod's spec contains `hostNetwork=true` (default is `false`), then its IP address isn't translated to a different address. This is the case for the kube-proxy and Amazon VPC CNI plugin for Kubernetes pods that run on your cluster, by default. For these pods, the IP address is the same as the node's primary IP address, so the pod's IP address isn't translated. For more information about a pod's hostNetwork setting, see PodSpec v1 core in the Kubernetes API reference.

```sh
kubectl exec --stdin --tty shell-demo -- /bin/bash
dig amazon.com
```

### **Lab-5 Monitoring and debugging**

#### Handle Liveness/Readiness Probe failures

We advise increasing the liveness and readiness probe timeout values (default timeoutSeconds: 10) for EKS 1.20 an later clusters to prevent probe failures from causing your application's Pod to become stuck in a containerCreating state. This problem has been seen in data-intensive and batch-processing clusters. High CPU use causes aws-node probe health failures, leading to unfulfilled Pod CPU requests. In addition to modifying the probe timeout, ensure that the CPU resource requests (default CPU: 25m) for aws-node are correctly configured. We do not suggest updating the settings unless your node is having issues.

#### Debugging for trouble ticket troubleshooting

We highly encourage you to run

```sh
sudo bash /opt/cni/bin/aws-cni-support.sh
```

on a node while you engage Amazon EKS support. The script will assist in evaluating kubelet logs and memory utilization on the node. Please consider installing SSM Agent on Amazon EKS worker nodes to run the script.

#### Monitor IP Address Inventory

You can monitor the IP addresses inventory of subnets using CNI Metrics Helper.

```sh
maximum number of ENIs the cluster can support
number of ENIs already allocated
number of IP addresses currently assigned to Pods
total and maximum number of IP address available
```

You can also set CloudWatch alarms to get notified if a subnet is running out of IP addresses. Please visit EKS user guide for install instructions of CNI metrics helper. Make sure `DISABLE_METRICS` variable for VPC CNI is set to `false`.

##### Creating a metrics dashboard

After you have deployed the CNI metrics helper, you can view the CNI metrics in the Amazon CloudWatch console.
**To create a CNI metrics dashboard**

1. Open the CloudWatch console at [https://console.aws.amazon.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/).
2. In the left navigation pane, choose **Metrics** and then select <strong>All metrics</strong>.
3. Choose the **Graphed metrics** tab.
4. Choose <strong>Add metrics using browse or query</strong>.
5. Make sure that under <strong>Metrics</strong>, you've selected the AWS Region for your cluster.
6. In the Search box, enter **Kubernetes** and then press <strong>Enter</strong>.
7. Select the metrics that you want to add to the dashboard.
8. At the upper right of the console, select <strong>Actions</strong>, and then <strong>Add to dashboard</strong>.
9. In the **Select a dashboard** section, choose <strong>Create new</strong>, enter a name for your dashboard, such as <strong>EKS-CNI-metrics</strong>, and then choose <strong>Create</strong>.
10. In the **Widget type** section, select <strong>Number</strong>.
11. In the **Customize widget title** section, enter a logical name for your dashboard title, such as <strong>EKS CNI metrics</strong>.
12. Choose **Add to dashboard** to finish. Now your CNI metrics are added to a dashboard that you can monitor. For more information about Amazon CloudWatch Logs metrics, see [Using Amazon CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) in the Amazon CloudWatch User Guide.

![Application logo](/image/image-11.png)

### End - Destroy the Lab environments

To teardown and remove the resources created in this example:

```sh
terraform destroy -target=kubectl_manifest.eni_config -target=module.eks_blueprints_kubernetes_addons -auto-approve
terraform destroy -target=module.eks -auto-approve
terraform destroy -auto-approve
```

## **All VPC CNI Use Cases:**

![Application logo](/image/image-12.png)
all use cases are described in referece : [https://docs.aws.amazon.com/eks/latest/userguide/pod-networking-use-cases.html](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking-use-cases.html)