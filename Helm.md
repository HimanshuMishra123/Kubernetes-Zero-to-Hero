## Detailed Notes on Kubernetes Helm

### Introduction to Kubernetes Package Management

**Concept Overview**
- **Package Management**: Simplifies the installation of software packages on your operating system.
- **Kubernetes Package Management(Helm)**: Manages mandatory installations such as monitoring setups (Prometheus, ELK stack) across various Kubernetes clusters using Helm.

**Why Helm?**
- **Community and CNCF Support**: Backed by a strong community and is part of the Cloud Native Computing Foundation (CNCF).
- **Ease of Use**: Simple to learn and use with excellent documentation.
- **Functional Benefits**: Supports upgrades and rollbacks, making it easy to manage different versions of applications.

### Helm Basics

**Key Components**
1. **Helm Chart**: A package that contains all the resource definitions necessary to run an application inside a Kubernetes cluster.
2. **Helm Repository**: A place to store and share charts.(Example - on Github)
3. **Helm Release**: An instance(example-ALB controller) of a chart running in a Kubernetes cluster.

### Creating and Using Helm Charts

**Creating a Helm Chart**
- Use `helm create <chart_name>` to generate the initial structure.
- Typical structure includes:
  - `templates/`: Contains Kubernetes resource files (e.g., Deployment, Service).
  - `values.yaml`: Contains default configuration values.
  - `Chart.yaml`: Metadata about the chart (name, version, description).
  - `charts/`: Holds dependencies for the chart.
  - `.helmignore`: Specifies files to be ignored.

  mychart/
  ├── Chart.yaml
  ├── values.yaml
  ├── charts/
  ├── templates/
  │   ├── _helpers.tpl
  │   ├── deployment.yaml
  │   ├── hpa.yaml
  │   ├── ingress.yaml
  │   ├── NOTES.txt
  │   ├── service.yaml
  │   └── serviceaccount.yaml
  └── .helmignore


**Example Commands**
- Create a Helm chart:
  ```sh
  helm create mychart
  ```
- Install a chart:
  ```sh
  helm install myrelease mychart
  ```

### Helm Chart Structure

**Files and Directories**
- **`templates/`**: Holds the actual YAML templates.
  - Example: `deployment.yaml`, `service.yaml`.
- **`values.yaml`**: Default configuration values.
  - Can be overridden during installation.
- **`Chart.yaml`**: Contains chart metadata.
- **`charts/`**: Manages chart dependencies.
- **`.helmignore`**: Ignores specified files when packaging.

### Managing Configurations

**Default and Custom Values**
- Default values are specified in `values.yaml`.
- Override values using:
  - Command line:
    ```sh
    helm install --set key=value myrelease mychart
    ```
  - Custom values file:
    ```sh
    helm install -f custom-values.yaml myrelease mychart
    ```

### Example Usage

**Simple Deployment Example**
1. Create a Helm chart:
   ```sh
   helm create helloworld
   ```
2. Modify `templates/deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: {{ .Values.deploymentName }}
   spec:
     replicas: {{ .Values.replicaCount }}
     selector:
       matchLabels:
         app: {{ .Values.appName }}
     template:
       metadata:
         labels:
           app: {{ .Values.appName }}
       spec:
         containers:
         - name: {{ .Values.appName }}
           image: {{ .Values.image }}
           ports:
           - containerPort: {{ .Values.containerPort }}
   ```
3. Modify `values.yaml`:
   ```yaml
   deploymentName: helloworld-deployment
   replicaCount: 2
   appName: helloworld
   image: nginx:1.14.2
   containerPort: 80
   ```
4. Install the chart:
   ```sh
   helm install helloworld-release ./helloworld
   ```

### Advanced Features

**Helm Hooks and Testing**
- **Helm Hooks**: Run scripts or jobs at specific points in the release lifecycle (e.g., pre-install, post-install).
- **Testing**: Helm allows running tests defined in the chart to verify the deployment.

**Example Commands**
- Test a release:
  ```sh
  helm test myrelease
  ```

### Helm Best Practices

**Best Practices for Helm Charts**
- **Use Readable Templates**: Keep templates simple and maintainable.
- **Document Values**: Provide clear documentation for `values.yaml`.
- **Versioning**: Follow semantic versioning for charts.
- **Security**: Regularly update charts and monitor for vulnerabilities.

### Common Interview Questions

**Helm 2 vs. Helm 3**
- **Helm 2**: Utilized a server component called Tiller.
- **Helm 3**: Removed Tiller, enhancing security by removing the need for cluster-wide permissions.
- **Architectural Differences**: Helm 3's architecture is more streamlined and secure.

### Practical Helm Commands

**Common Commands**
- **Search for Charts**:
  ```sh
  helm search hub <keyword>
  ```
- **Add Repository**:
  ```sh
  helm repo add <repo_name> <repo_url>
  ```
- **Update Repositories**:
  ```sh
  helm repo update
  ```
- **List Releases**:
  ```sh
  helm list
  ```

### Conclusion

Helm simplifies Kubernetes application management through reusable, configurable, and shareable packages called Helm charts. Its robust community support, ease of use, and comprehensive features make it a preferred choice for Kubernetes package management. By mastering Helm, you can streamline application deployments, upgrades, and rollbacks in your Kubernetes clusters.