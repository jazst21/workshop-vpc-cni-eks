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