# Create a kubeconfig file

You can add a cluster to Astra Control Service using a kubeconfig file. Depending on the type of cluster you want to add, you might need to manually create a kubeconfig file for your cluster using specific steps.
- [Create a kubeconfig file for Amazon EKS clusters](#Create-a-kubeconfig-file-for-Amazon-EKS-clusters)
- [Create a kubeconfig file for other types of clusters](#Create-a-kubeconfig-file-for-other-types-of-clusters)

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

3. Apply the service account:
    ```
    kubectl apply -f astracontrol-service-account.yaml
    ```

4. Create a ```ClusterRoleBinding``` file called ```astracontrol-clusterrolebinding.yaml```.

    ```
    astracontrol-clusterrolebinding.yaml
    ```
    ```apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: astra-admin-binding
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: astra-admin-account
      namespace: kube-system
    ```

5. Apply the cluster role binding:

    ```
    kubectl apply -f astracontrol-clusterrolebinding.yaml
    ```

6. Create a service account token secret file called ```astracontrol-secret.yaml```.

    ```
    astracontrol-secret.yaml
    ```
    ```
    apiVersion: v1
    kind: Secret
    metadata:
      annotations:
        kubernetes.io/service-account.name: astra-admin-account
      name: astra-admin-account
      namespace: kube-system
    type: kubernetes.io/service-account-token
    ```

7. Apply the token secret:

    ```
    kubectl apply -f astracontrol-secret.yaml
    ```

8. Retrieve the token secret:

    ```
    kubectl get secret astra-admin-account -n kube-system -o jsonpath='{.data.token}' | base64 -d
    ```

9. Replace the user section of the AWS EKS kubeconfig file with the token, as shown in the following example:

    ```
    user:
        token: k8s-aws-v1.aHR0cHM6Ly9zdHMudXMtd2VzdC0yLmFtYXpvbmF3cy5jb20vP0FjdGlvbj1HZXRDYWxsZXJJZGVudGl0eSZWZXJzaW9uPTIwMTEtMDYtMTUmWC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBM1JEWDdKU0haWU9LSEQ2SyUyRjIwMjMwNDAzJTJGdXMtd2VzdC0yJTJGc3RzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyMzA0MDNUMjA0MzQwWiZYLUFtei1FeHBpcmVzPTYwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCUzQngtazhzLWF3cy1pZCZYLUFtei1TaWduYXR1cmU9YjU4ZWM0NzdiM2NkZGYxNGRhNzU4MGI2ZWQ2zY2NzI2YWIwM2UyNThjMjRhNTJjNmVhNjc4MTRlNjJkOTg2Mg
    ```

## Create a kubeconfig file for other types of clusters
Follow these instructions to create a kubeconfig file for Rancher, Upstream Kubernetes, and Red Hat OpenShift clusters.

### Before you begin
Ensure that you have the following on your machine before you start:
- kubectl v1.23 or later installed
- An active kubeconfig with cluster admin rights for the active context

### Steps
1. Create a service account as follows:

    a. Create a service account file called ```astracontrol-service-account.yaml```.
    Adjust the name and namespace as needed. If changes are made here, you should apply the same changes in the following steps.

        ```
        astracontrol-service-account.yaml
        ```
        ```
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: astracontrol-service-account
          namespace: default
        ```

    b. Apply the service account:
    
    ```
    kubectl apply -f astracontrol-service-account.yaml
    ```


