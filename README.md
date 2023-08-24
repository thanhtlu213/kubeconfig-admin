# Create a kubeconfig file

You can add a cluster to Astra Control Service using a kubeconfig file. Depending on the type of cluster you want to add, you might need to manually create a kubeconfig file for your cluster using specific steps.
- Create a kubeconfig file for Amazon EKS clusters
- reate a kubeconfig file for other types of clusters

## Create a kubeconfig file for Amazon EKS clusters
Follow these instructions to create a kubeconfig file and permanent token secret for Amazon EKS clusters. A permanent token secret is required for clusters hosted in EKS.

### Steps
1. Follow the instructions in the Amazon documentation to generate a kubeconfig file:
[Creating or updating a kubeconfig file for an Amazon EKS cluster](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html)

2. Create a service account as follows:
a. Create a service account file called ```astracontrol-service-account.yaml```.
Adjust the service account name as needed. The namespace kube-system is required for these steps. If you change the service account name here, you should apply the same changes in the following steps.
    ```
    astracontrol-service-account.yaml
    ```
    ```
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: astra-admin-account
      namespace: kube-system
    ```


