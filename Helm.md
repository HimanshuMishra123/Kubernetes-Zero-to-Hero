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
  - `templates/`: Contains Kubernetes resource files (- Example: `deployment.yaml`, `service.yaml`.).
  - `values.yaml`: Contains default configuration values.(inhe override/change kro and chart use kro as per your need)
  - `Chart.yaml`: Metadata about the chart (name, version, description).
  - `charts/`: This directory is used to hold any dependencies that your chart might have. Dependencies are other charts that this chart relies on. Initially this is empty.
  - `.helmignore`: Specifies files and  directories that should be ignored when packaging the chart.

### Helm Chart Structure
Executing helm create mychart will create a directory called mychart with the following structure:
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

In above helm install command (to install packages/softwares on kubernetes cluster) <br/>
myrelease - This is the name you are giving to this Helm release(Instance running on k8s cluster). <br/>
mychart - This specifies the Helm chart to be installed. <br/>


### Managing Configurations

**Default and Custom Values**
- Default values are specified in `values.yaml`.
- Override values using:
  - Command line:
    ```sh
    helm install --set key=value myrelease mychart
    ```
  - Custom values file(recommended*):
    ```sh
    helm install -f custom-values.yaml myrelease mychart
    ```

### Example Usage

**Simple Deployment Example**
1. Create a Helm chart with name "helloworld":
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
   or

   ```sh
   helm install <release_name> <repo_name>/<chart_name>
   ```


### Practical Helm Commands

**Common Commands**
- **Search for Charts**:To search for list of charts(packages) in your added repositories url
  ```sh
  helm search hub <keyword>
  ```
- **Add Repository to local**:
  ```sh
  helm repo add <repo_name> <repo_url>
  ```

repo_name: A name(for local) you assign to the repository for reference.<br/>
repo_url: The URL of the Helm repository you are adding.<br/>
helm repo add command does not download the entire content of the charts in the repository. Instead, it simply registers the repository with your local Helm client by adding its metadata (such as the repository URL) to the repositories.yaml file located in ~/.config/helm (on Linux/macOS) or %USERPROFILE%\AppData\Roaming\helm (on Windows).<br/>
Below is repositeries.yaml<br/>
```
apiVersion: v1
repositories:
  - name: stable
    url: https://charts.helm.sh/stable        #Stable charts repositery
  - name: bitnami
    url: https://charts.bitnami.com/bitnami   #Bitnami charts repositery
  - name: myrepo(your custom made repo)
    url: https://example.com/helm-charts
```

To install packages using helm(first do helm search then below)-
```
# Install a chart from the Bitnami repository
helm install my-mysql bitnami/mysql   

# Install a chart from the official Helm repository
helm install my-nginx stable/nginx

# Install a chart from your custom internal repository
helm install my-custom-app myrepo/custom-app
```
If you are not specifying the repo name for package to be installed as above then helm will lookup the package in order the repositeries are added and whereever it finds the package first helm will install it. Example- helm install my-mysql mysql 


##### Either you create your own helm in local using helm create and then use helm install to install the packages (or) you do helm repo add to first add(register) the repository and then do helm repo install to install it.<br/>


- **Update Repositories**: This will update the repo
  ```sh
  helm repo update <repo_name>
  ```
- **List Releases**: This command reads from repositories.yaml and displays the list of repositories you have added.
  ```sh
  helm repo list
  ```
  ```
  output of helm repo list:
  NAME    URL
  myrepo  https://example.com/helm-charts
  stable  https://charts.helm.sh/stable
  ```

- **Remove Repositories**: This command will remove the specified repository from repositories.yaml.
  ```sh
  helm repo remove <repo_name>
  ```

   
### Advanced Features

**Helm Hooks and Testing**
- **Helm Hooks**: Run scripts or jobs at specific points in the release lifecycle (e.g., pre-install, post-install,upgrade, delete).They are useful for tasks such as data migrations, cleanup, or validation checks.
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
- **Helm 2**: Utilized a server component called Tiller(responsible for managing releases within the Kubernetes cluster).
- **Helm 3**: Removed Tiller, enhancing security by removing the need for cluster-wide permissions(which was needed for tiller).
- **Architectural Differences**: Helm 3's architecture is more streamlined and secure.

### Benefits
**Standardization**: Provides a standardized structure to start with, making it easier to organize and manage your Kubernetes resources.
**Customization**: You can customize the templates and values as per your application's requirements.
**Reuse**: The generated chart can be reused and shared with others, promoting consistency and efficiency in deploying applications.

### Conclusion

Helm simplifies Kubernetes application management through reusable, configurable, and shareable packages called Helm charts. Its robust community support, ease of use, and comprehensive features make it a preferred choice for Kubernetes package management. By mastering Helm, you can streamline application deployments, upgrades, and rollbacks in your Kubernetes clusters.