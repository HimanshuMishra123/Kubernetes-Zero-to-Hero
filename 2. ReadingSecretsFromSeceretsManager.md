## Reading Secrets dynamically in EKS as volume mounts From AWS Secerets Manager

To read secrets from AWS Secrets Manager in an Amazon EKS deployment, you can integrate Secrets Manager with Kubernetes using the **AWS Secrets and Configuration Provider (ASCP)**, or through **EKS service account IAM roles** with **Kubernetes secrets**.

Here’s a step-by-step guide on how to achieve this:

### Method 1: Using AWS Secrets and Configuration Provider (ASCP) via `Secrets Store CSI Driver`

The **Secrets Store CSI Driver** allows you to mount secrets, configuration, and certificates stored in external secret management systems into Kubernetes pods as files.

#### 1. **Install the Secrets Store CSI Driver and AWS Provider**
   
   Install the **Secrets Store CSI Driver** and **AWS Secrets Provider**:

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/main/deploy/rbac-secretproviderclass.yaml
   kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/main/deploy/csidriver.yaml
   kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-secrets-and-config-provider/main/deployment/aws-provider-installer.yaml
   ```

#### 2. **Create a SecretProviderClass for your secret**

   This resource will define the integration between Secrets Manager and Kubernetes.

   ```yaml
   apiVersion: secrets-store.csi.x-k8s.io/v1
   kind: SecretProviderClass
   metadata:
     name: aws-secrets
     namespace: default
   spec:
     provider: aws
     parameters:
       objects: |
         - objectName: "my-secret"    # AWS Secrets Manager secret name
           objectType: "secretsmanager"
   ```

   The `objectName` is the name of the secret in AWS Secrets Manager that you want to use.

#### 3. **Modify your `deployment.yml` to use the secret**

   Now, in your EKS deployment, you can mount the secret using the **CSI driver**:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-app
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-app
           image: my-app-image
           volumeMounts:
           - name: secrets-store-inline
             mountPath: "/mnt/secrets"    # Mount path for secrets
             readOnly: true
         volumes:
         - name: secrets-store-inline
           csi:
             driver: secrets-store.csi.k8s.io
             readOnly: true
             volumeAttributes:
               secretProviderClass: "aws-secrets"
   ```

   - **Mount Path**: The secrets will be available at `/mnt/secrets`.
   - **CSI Driver**: This connects the secret store to your deployment.

#### 4. **IAM Role for Service Account (IRSA) Configuration**

Ensure the service account for the deployment has permissions to read from AWS Secrets Manager. You will need to create an IAM role with appropriate permissions and associate it with the Kubernetes service account using IRSA (IAM Roles for Service Accounts).

**IAM Policy**:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue"
            ],
            "Resource": "arn:aws:secretsmanager:region:account-id:secret:my-secret"
        }
    ]
}
```

Attach this policy to the IAM role that will be assumed by your EKS pod service account.

