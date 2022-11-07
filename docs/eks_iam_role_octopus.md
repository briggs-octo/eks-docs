# EKS Internal Role-Based Authentication ![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)

## AWS Identity and Access Management console (IAM) 

1. Create [the following custom IAM policy](https://github.com/briggs-octo/eks-docs/blob/main/aws/iam/eks_full_access.json) named `eks_full_access` which will allow the IAM role created in step to 2 to acess EKS.
2. Create a new IAM role for accessing EKS named `iam_eks_service_role`, attaching the IAM policy create in step 1.
3. Set [the following Trusted entities](https://github.com/briggs-octo/eks-docs/blob/main/aws/iam/eks_internal_role_trusted_entities.json) via the `Trusted relationships` tab within the EKS service role created in step 1 (`iam_eks_service_role`).

---
**NOTE**

This section below assumes the following:

- The EKS cluster we want to manage via the IAM role `iam_eks_service_role` is named `client_cluster`.
- The IAM user/role has access to `client_cluster`.
- The correct version of kubectl is installed.
  
---

## Command Line (kubectl)

1. Configure kubectl so that it can connect to `client_cluster`.
   1. `aws eks update-kubeconfig --name client_cluster --region <region>`
2. Apply [the following Kubernetes Config Map yaml](https://github.com/briggs-octo/eks-docs/blob/main/kubernetes/eks_internal_role_config_map.yml) to the cluster.
   1. `kubectl apply -f eks_role_config_map.yml`
3. Describe the Config Map to confirm new values exist.
   1. `kubectl describe configmap -n kube-system aws-auth`
