# EKS Cross-Account Role-Based Authentication ![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)

## Context

By default, EKS resources are visible only to the role that created the cluster. If you've changed your IAM structure, or need to enable cross-account visibility (e.g. in the instance of a shared services AWS account), you need to take additional steps both at the AWS authentication and the underlying EKS levels to enable appropriate visibility.

This guide shows the required steps assuming the following scenario:

* There is a _client AWS account_ that has a preexisting EKS cluster
* There is a _management AWS account_ that wants to be able to access and read the information about the EKS resources

## AWS Identity and Access Management console (IAM) (Client Account)

1. Create [the following custom IAM policy](https://github.com/briggs-octo/eks-docs/blob/main/aws/iam/eks_full_access.json) named `eks_full_access` .
2. Create a new IAM role (AWS Account Entity Type) named `iam_management_eks_role` that will allow the management account to access the EKS cluster in the client account.
   1. Supply the management account ID within the role.
   2. Attach the IAM policy created in step 1 to this role (`eks_full_access`).
3. Ensure that the Trusted Entities for the `iam_management_eks_role` IAM role created in step 2 (visible via the `Trusted relationships` tab) contains [the following Trusted entities](https://github.com/briggs-octo/eks-docs/blob/main/aws/iam/eks_management_role_trusted_entities.json).

## Test Cross-Account Configuration (AWS Console) (Management Account)

1. Log into the management AWS account using an IAM user with the ability to assume the role ([example policy](https://github.com/briggs-octo/eks-docs/blob/main/aws/iam/management_assume_role.json)).
2. Assume the role created in section 1 above via the right-hand account drop-down in the AWS Web UI (Switch Role button).
   1. Use the client account Id for the Account value and `iam_management_eks_role` for the Role value.  
3. Navigate to the EKS service to verify that you can see the client account's cluster(s).
4. Click into a client cluster.
   1. A blue RBAC warning/error banner should display and the cluster's reosurces shoud not be visible.
5. Proceed to the next section to configure EKS RBAC permissions.

---
**NOTE**

The section below assumes the following:

- The EKS cluster we want to manage via the IAM role `iam_management_eks_service_role` is named `client_cluster`.
- The IAM user/role has access to run commands against `client_cluster`.
- The correct version of kubectl is installed.
  
---

## Command Line (kubectl)

1. Configure kubectl so that it can connect to `client_cluster`.
   1. `aws eks update-kubeconfig --name client_cluster --region <AWS_REGION>`
2. Apply [the following Config Map yaml](https://github.com/briggs-octo/eks-docs/blob/main/kubernetes/eks_management_config_map.yml) to `client cluster`.
   1. `kubectl apply -f eks_management_config_map.yml`

## Octopus Deploy EKS Configuration (Optional)

1. Configure a Kubernetes deploy target in Octopus Deploy.
   1. Use a Windows dynamic worker and run within the `octopusdeploy/worker-tools:3.3.2-windows.ltsc2019` container. image for health checks.
   2. Use the ARN for the management role created in the client account for `Assumed Role ARN` under authentication.
   3. Specify any value for the `Assumed Role Session Name` under authentication.
2. Run a health check against the new EKS target.
