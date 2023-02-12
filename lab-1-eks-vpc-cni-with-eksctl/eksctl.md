Prerequisites

    New or Existing EKS Cluster. You can follow the guidelines documented here to create an EKS cluster using the portal or eksctl.

    Amazon VPC CNI (v1.9.0 or later) add-on added to your cluster. To add or update the add-on on your cluster, see Managing the Amazon VPC CNI add-on. If you want to update the version in the existing cluster, run the below command.

    eksctl update addon \
     --name vpc-cni \
     --version <1.9.x-eksbuild.y> \
     --cluster <my-cluster> \
     --force

    Amazon EC2 Nitro instances for node groups in the cluster.

How to enable the feature?

To enable the IP prefix assignment feature, run the below command.

kubectl set env daemonset aws-node \
 -n kube-system ENABLE_PREFIX_DELEGATION=true

Calculate the number of maximum pods per node.

Download the max pods per node calculator script and change the access mode.

curl -o max-pods-calculator.sh https://raw.githubusercontent.com/awslabs/amazon-eks-ami/master/files/max-pods-calculator.sh
chmod +x max-pods-calculator.sh

Calculate the max pods using the max pods calculator script. You can pass the instance type and CNI version to calculate the max pods per node.

./max-pods-calculator.sh \
--instance-type <m5.large> \
--cni-version <1.9.x-eksbuild.y> \
--cni-prefix-delegation-enabled

You can take a look at the max pods calculator script. Without enabling the prefix delegation feature or non-nitro instance types, the calculation of the pods are ENI * (# of IPv4 per ENI - 1) + 2.

The calculation changed into ENI ((# of IPv4 per ENI - 1) 16) + 2 where 16 is IPS_PER_PREFIX. The pod density per node is significantly more than before.
Create Node Groups with EC2 Nitro Amazon Linux 2 Instance Type

Create one of the following node groups with at least one Amazon EC2 Nitro Amazon Linux 2 instance type. See the list of Nitro Amazon Linux 2 Instance Types.

eksctl create nodegroup \
  --cluster <my-cluster> \
  --region <us-east-1> \
  --name <my-nodegroup> \
  --node-type <m5.large> \
  --managed \
  --max-pods-per-node <110> -- Count from the max pods calculator

Once your nodes are deployed, you can run the kubectl describe command in the node to see the max pods in the allocatable.

eksctl delete cluster eks-vpc-cni-eksctl-1
