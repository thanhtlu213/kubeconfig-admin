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

2. Grant cluster admin permissions as follows:

    a. Create a ClusterRoleBinding file called astracontrol-clusterrolebinding.yaml.
    
    Adjust any names and namespaces modified when creating the service account as needed.
    ```
    astracontrol-clusterrolebinding.yaml
    ```
    ```
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: astracontrol-admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: astracontrol-service-account
      namespace: default    
    ```

    b. Apply the cluster role binding:
    ```
    kubectl apply -f astracontrol-clusterrolebinding.yaml
    ```

3. List the service account secrets, replacing ```context``` with the correct context for your installation:
    ```
    kubectl get serviceaccount astracontrol-service-account --context <context> --namespace default -o json
    ```

    The end of the output should look similar to the following:
    ```
    "secrets": [
    { "name": "astracontrol-service-account-dockercfg-vhz87"},
    { "name": "astracontrol-service-account-token-r59kr"}
    ]    
    ```
    The indices for each element in the ```secrets``` array begin with 0. In the above example, the index for ```astracontrol-service-account-dockercfg-vhz87``` would be 0 and the index for ```astracontrol-service-account-token-r59kr``` would be 1. In your output, make note of the index for the service account name that has the word "token" in it.

4. Generate the kubeconfig as follows:

    a. Create a ```create-kubeconfig.sh``` file. Replace ```TOKEN_INDEX``` in the beginning of the following script with the correct value.
    ```
    create-kubeconfig.sh
    ```
    ```
    # Update these to match your environment.
    # Replace TOKEN_INDEX with the correct value
    # from the output in the previous step. If you
    # didn't change anything else above, don't change
    # anything else here.

    SERVICE_ACCOUNT_NAME=astracontrol-service-account
    NAMESPACE=default
    NEW_CONTEXT=astracontrol
    KUBECONFIG_FILE='kubeconfig-sa'

    CONTEXT=$(kubectl config current-context)

    SECRET_NAME=$(kubectl get serviceaccount ${SERVICE_ACCOUNT_NAME} \
      --context ${CONTEXT} \
      --namespace ${NAMESPACE} \
      -o jsonpath='{.secrets[TOKEN_INDEX].name}')
    TOKEN_DATA=$(kubectl get secret ${SECRET_NAME} \
      --context ${CONTEXT} \
      --namespace ${NAMESPACE} \
      -o jsonpath='{.data.token}')

    TOKEN=$(echo ${TOKEN_DATA} | base64 -d)

    # Create dedicated kubeconfig
    # Create a full copy
    kubectl config view --raw > ${KUBECONFIG_FILE}.full.tmp

    # Switch working context to correct context
    kubectl --kubeconfig ${KUBECONFIG_FILE}.full.tmp config use-context ${CONTEXT}

    # Minify
    kubectl --kubeconfig ${KUBECONFIG_FILE}.full.tmp \
      config view --flatten --minify > ${KUBECONFIG_FILE}.tmp

    # Rename context
    kubectl config --kubeconfig ${KUBECONFIG_FILE}.tmp \
      rename-context ${CONTEXT} ${NEW_CONTEXT}

    # Create token user
    kubectl config --kubeconfig ${KUBECONFIG_FILE}.tmp \
      set-credentials ${CONTEXT}-${NAMESPACE}-token-user \
      --token ${TOKEN}

    # Set context to use token user
    kubectl config --kubeconfig ${KUBECONFIG_FILE}.tmp \
      set-context ${NEW_CONTEXT} --user ${CONTEXT}-${NAMESPACE}-token-user

    # Set context to correct namespace
    kubectl config --kubeconfig ${KUBECONFIG_FILE}.tmp \
      set-context ${NEW_CONTEXT} --namespace ${NAMESPACE}

    # Flatten/minify kubeconfig
    kubectl config --kubeconfig ${KUBECONFIG_FILE}.tmp \
      view --flatten --minify > ${KUBECONFIG_FILE}

    # Remove tmp
    rm ${KUBECONFIG_FILE}.full.tmp
    rm ${KUBECONFIG_FILE}.tmp
    ```
    
    b. Source the commands to apply them to your Kubernetes cluster.
    ```
    source create-kubeconfig.sh
    ```


5. (Optional) Rename the kubeconfig to a meaningful name for your cluster. Protect your cluster credential.

    ```
    chmod 700 create-kubeconfig.sh
    mv kubeconfig-sa YOUR_CLUSTER_NAME_kubeconfig
    ```
