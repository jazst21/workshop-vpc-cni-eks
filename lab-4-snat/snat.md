SNAT for pods

If you deployed your cluster using the IPv6 family, then the information in this topic isn't applicable to your cluster, because IPv6 addresses are not network translated. For more information about using IPv6 with your cluster, see Tutorial: Assigning IPv6 addresses to pods and services.

By default, each pod in your cluster is assigned a private IPv4 address from a classless inter-domain routing (CIDR) block that is associated with the VPC that the pod is deployed in. Pods in the same VPC communicate with each other using these private IP addresses as end points. When a pod communicates to any IPv4 address that isn't within a CIDR block that's associated to your VPC, the Amazon VPC CNI plugin for Kubernetes

translates the pod's IPv4 address to the primary private IPv4 address of the primary elastic network interface of the node that the pod is running on, by default *.

Due to this behavior:

    Resources that are in networks or VPCs that are connected to your cluster VPC using VPC peering, a transit VPC, or Direct Connect can't initiate communication to your pods. Your pods can initiate communication to those resources and receive responses from them though.

    Your pods can communicate with internet resources only if the node that they're running on has a public or elastic IP address assigned to it and is in a public subnet. A public subnet's associated route table has a route to an internet gateway. We recommend deploying nodes to private subnets, whenever possible.

If you have resources in networks or VPCs that are connected to your cluster VPC using VPC peering, a transit VPC, or Direct Connect that need to initiate communication with your pods using an IPv4 address, then you need to change the default configuration with the following command.

```sh
kubectl set env daemonset -n kube-system aws-node AWS_VPC_K8S_CNI_EXTERNALSNAT=true
```
If you've changed the setting to true and want your pods to communicate to the internet, then the route table that's associated with the private subnet that your node is deployed in must contain a route to a public NAT gateway.

*If a pod's spec contains `hostNetwork=true` (default is `false`), then its IP address isn't translated to a different address. This is the case for the kube-proxy and Amazon VPC CNI plugin for Kubernetes pods that run on your cluster, by default. For these pods, the IP address is the same as the node's primary IP address, so the pod's IP address isn't translated. For more information about a pod's hostNetwork setting, see PodSpec v1 core
in the Kubernetes API reference. 

```sh
kubectl exec --stdin --tty shell-demo -- /bin/bash
dig amazon.com
```