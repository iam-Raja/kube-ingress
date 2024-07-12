### EKS Ingress-AWS Load Balancer Controller installation ###

* 1.Create an IAM OIDC provider. You can skip this step if you already have one for your cluster.
```
eksctl utils associate-iam-oidc-provider \
    --region <region-code> \
    --cluster <your-cluster-name> \
    --approve
```

* 2.Download an IAM policy for the LBC using one of the following commands:

If your cluster is in a US Gov Cloud region:
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.8.1/docs/install/iam_policy.json
```


* 3.Create an IAM policy named AWSLoadBalancerControllerIAMPolicy. If you downloaded a different policy, replace iam-policy with the name of the policy that you downloaded.
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```


* 4.Create an IAM role and Kubernetes ServiceAccount for the LBC. Use the ARN from the previous step.
```
eksctl create iamserviceaccount \
--cluster=<cluster-name> \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region <region-code> \
--approve
```
**Add controller to cluster**

* 1.Helm install command for clusters with IRSA:
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<cluster-name> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```