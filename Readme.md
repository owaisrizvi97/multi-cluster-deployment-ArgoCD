# Multi Cluster Application Deployment using ArgoCD (Hub-spoke Model)

# prerequisites

kubectl – A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl.
https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

eksctl – A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.
https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html

AWS CLI – A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI
https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html in the AWS Command Line Interface User Guide. 

After installing the AWS CLI, I recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide.
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config 

Argo CD CLI(Not the actual Argo CD Installation) - 
https://argo-cd.readthedocs.io/en/stable/cli_installation/#installation

# EKS Setup

## EKS Clusters Creation

```
eksctl create cluster --name hub-cluster --region us-west-1
```
```
eksctl create cluster --name spoke-cluster-1 --region us-west-1
```
```
eksctl create cluster --name spoke-cluster-2 --region us-west-1
```

![5 - glgCIZo](https://github.com/owaisrizvi97/multi-cluster-deployment-ArgoCD/assets/68285890/75bec29b-888f-4783-b20a-128b57a5e297)


![6 - 7mEJFoi](https://github.com/owaisrizvi97/multi-cluster-deployment-ArgoCD/assets/68285890/07d75605-4bb2-4818-b2ed-8769ed744cb0)



## Kubernetes context connectivity with EKS

```
kubectl config get-contexts
```
```
kubectl config use-context <context-name>
```
## Install ArgoCD

```
kubectl create namespace argocd
```

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

![2 - TJMr7lU](https://github.com/owaisrizvi97/multi-cluster-deployment-ArgoCD/assets/68285890/0dbd6d74-a37a-424c-93e7-19a0622249df)


## Run Argo CD in HTTP Mode(Insecure)

https://github.com/argoproj/argo-cd/blob/54f1572d46d8d611018f4854cf2f24a24a3ac088/docs/operator-manual/argocd-cmd-params-cm.yaml#L82

## Expose Argo CD Server Service in NodePort Mode

```
kubectl edit svc argocd-server -n argocd
```

and change the type to NodePort from ClusterIP

```
kubectl get svc argocd-server -n argocd
```
![7 - u4xfMhn](https://github.com/owaisrizvi97/multi-cluster-deployment-ArgoCD/assets/68285890/ee13742e-b1d9-40b1-b51d-dbb098014d7a)


## Login to ArgoCD 

Hit the Hub-cluster nodeport IP (two Ec2 instance is running, we can use public IP of any of our choice) with specified port in previous command output.

username is admin, to get password. Run:

```
kubectl get secret -n argocd
```

```
kubectl edit secret argocd-initial-admin-secret -n argocd
```

Copy the secret & decode it's base64 format:

```
echo [unique-encrypted-token] | base64 --decode
```

except for % sign, use it as your pass.

## Login to argocd-cli

```
argocd login 13.233.140.21:32002
```

## Add Spoke cluster 1 & spoke cluster 2 in Hub Cluster (or ArgoCD server Cluster)

```
argocd cluster add devops-project@spoke-cluster-1.ap-south-1.eksctl.io --server 13.233.140.21:32002
```

```
argocd cluster add devops-project@spoke-cluster-2.ap-south-1.eksctl.io --server 13.233.140.21:32002
```

## Add ArgoCD Applications & test with manual changes


![1 - SWMymF0](https://github.com/owaisrizvi97/multi-cluster-deployment-ArgoCD/assets/68285890/b13e222f-3fee-4e30-8c20-a73a9a2c02e1)

![3 - CXtARwQ](https://github.com/owaisrizvi97/multi-cluster-deployment-ArgoCD/assets/68285890/b50f2186-f12d-49cf-9ca4-eb90de43af0d)


![4 - 27xYyuw](https://github.com/owaisrizvi97/multi-cluster-deployment-ArgoCD/assets/68285890/2c2065cb-b82e-44a7-b563-e8afc1f120ff)




# EKS Clusters Deletion

eksctl delete cluster --name hub-cluster --region us-west-1

eksctl delete cluster --name spoke-cluster-1 --region us-west-1

eksctl delete cluster --name spoke-cluster-2 --region us-west-1
