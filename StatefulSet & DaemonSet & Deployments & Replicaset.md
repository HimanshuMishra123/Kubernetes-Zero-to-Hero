### StatefulSet vs. DaemonSet vs. Deployments vs. ReplicaSet

![Difference-between-ReplicaSet-ReplicationController-2](https://github.com/user-attachments/assets/977d2848-22bf-414f-833f-f15422cf0eb7)

![statefulSets-in-kubernetes-ezgif com-webp-to-jpg-converter](https://github.com/user-attachments/assets/15d7a0d6-4a91-480f-8986-4316990d89b9)

![kubernetes-statefulset](https://github.com/user-attachments/assets/fa82cc2d-e7e3-44d6-a794-b4609fe79ddd)



How stateful apps are different than stateless apps? As we will see, the `state` aspect of stateful apps makes orchestration more complex than what the initial Kubernetes controllers were built for.

**Stateful Applications** are applications that are mindful of their past and present state. In a nutshell, the applications that keep track of their state. They store data using persistent storage and read the data later to survive service breakdown or restarts. Database applications like MySQL , MongoDB, and distributed systems like Apache Kafka are examples of stateful applications. StatefulSet is the controller that manages the deployment and scaling of a set of Stateful application pods. A stateful pod in Kubernetes is a pod that requires persistent storage and a stable network identity to maintain its state all the time, even during pod restarts or rescheduling. Unlike a Deployment, which manages stateless applications, a StatefulSet maintains a unique identity for each of its pods and ensures that they are deployed in a predictable order.

On the other hand, **Stateless Applications** are applications that do not monitor any of their states. They neither store nor read data from any storage; they are basically a one-time request and feedback process. When a stateless application’s current session is down, interrupted or deleted, the new session will start with a clean slate, without referring to past events or processes. Examples include the Nginx web application and Tomcat web server.

### Voulme attachment
![c0e51-1jcqz09xnlqufsm3sl-ixvw](https://github.com/user-attachments/assets/8b473244-fd2f-46b8-8085-c7bf6d760556)

As highlighted in Containerizing Stateful Applications, data volume mapped to a container can be either:

1. Inside the container

2. Inside the host, independent of the container

3. Outside the host, independent of both the host and the container

Stateless applications by definition have no need for long-running persistence and hence usually rely on the first 2 options. However, stateful applications (such as databases, message queues, metadata stores) have to use the 3rd approach in order to guarantee that the data outlives the lifecycle of a container or a host. The above also means that as new containers or new hosts are brought up, we should be able to map them to existing persistent volumes.


![20190808_162532179_90804 (1)](https://github.com/user-attachments/assets/36579d94-b60f-4af9-9d13-25e232db263f)
#### Key Features of Statefulset:
- **Stable and unique network identity:** Each pod in a StatefulSet has a unique, stable network identity that persists across rescheduling.
- **Stable storage(PV):** StatefulSets can provide stable storage using PersistentVolumes (PVs) to the pods, ensuring that each pod has a persistent volume even if it is rescheduled. Each pod created by a StatefulSet is associated with a unique PersistentVolumeClaim (PVC), allowing the pod to access the same storage if it gets rescheduled or restarted even  to a different node as well.
- **Ordered deployment and scaling:** Pods are created in sequential order, and scaling operations are performed in a controlled manner. StatefulSets support both manual and automatic scaling of pods. It provide guarantees about the order in which pods are created, updated, and deleted. When you scale up a StatefulSet, each pod is created in a specific order, and when you scale down, pods are terminated in the reverse order. This ordering ensures that data consistency and dependencies between pods are maintained.
- **Ordered, automated rolling updates:** Rolling updates can be performed on a StatefulSet, ensuring zero-downtime updates.
- **Headless Service:** StatefulSets automatically create a headless service for accessing individual pods. The headless service allows direct communication with each pod by its hostname. It can be used for peer discovery or connecting to specific instances in the StatefulSet.
- **StatefulSet Controller:** The StatefulSet controller monitors the desired state and actual state of pods. It ensures that the number of pods matches the desired replica count. The controller handles pod creation, deletion, scaling, and rolling updates.

#### Common Use Cases statefulset:
- Databases (e.g., MySQL, PostgreSQL)
- Distributed systems (e.g., Kafka, Zookeeper)
- Stateful applications requiring stable, persistent storage and network identities

### PersistentVolume (PV)
A PersistentVolume (PV) is a piece of storage in a Kubernetes cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. PVs are independent of the pods that use them, making them durable and persistent.

#### Key Features of PV:
- **Provisioning:** PVs can be statically created by an administrator or dynamically provisioned using StorageClasses.
- **Reclaim Policy:** When a user is done with a PV, it can either be retained, recycled, or deleted based on the reclaim policy.
- **Access Modes:** PVs support different access modes, including ReadWriteOnce, ReadOnlyMany, and ReadWriteMany, dictating how the volume can be mounted by pods.
- **PersistentVolumeClaims (PVCs):** Users request PVs by creating PersistentVolumeClaims (PVCs). The cluster then creates and binds the PVC to a suitable PV, ensuring the user gets the requested storage. on a high level, When a PVC is created with a reference to the Storage Class, the CSI contoller Plugin detects the PVC automatically and use storage class (SC) info to provisions an persistent volume and attaches it to the StatefulSet pod.

#### Common Use Cases PV:
- Storing application data
- Maintaining logs
- Providing shared storage for stateful applications

Read  example of reddis as statefulset with PV from... 
https://github.com/HimanshuMishra123/three-tier-architecture-demo/blob/master/self-readme.md

![Capture](https://github.com/user-attachments/assets/7bf14a42-dc2c-4231-af95-7832370b749d)
---

## DaemonSet vs. Deployments vs. ReplicaSet

![Difference-between-ReplicaSet-ReplicationController-2](https://github.com/user-attachments/assets/977d2848-22bf-414f-833f-f15422cf0eb7)

![6eac4-1fhhtm36vopagnytud1wdjq](https://github.com/user-attachments/assets/e6946205-af4d-4edb-be78-5b2e839003d6)
Common Kubernetes Controller APIs Not Fit For Stateful Apps. none of the above controllers are suitable for running stateful apps. Hence, the need for StatefulSets, a special controller purpose-built for the needs of stateful apps.

**Deployment** — Manages stateless applications and extends the ReplicaSet functionality to add update semantics that allows pod recreation and rolling upgrades and scaling.

**ReplicaSet** — Ensures a specified number of pod replicas are running but with ephemeral storage only. No support for persistent storage so clearly not a fit for stateful apps. Used by Deployments to manage pods but can be used independently for simpler scenarios.

**Job** — used for running batch jobs at a pre-determined schedule but it does not handle scheduling on its own. For running jobs at a pre-determined schedule, you would use a CronJob(short-lived, one-time tasks). Here are detailed descriptions of both Job and CronJob:

### Job
A Job in Kubernetes creates one or more pods and ensures that a specified number of them successfully complete. Once a job is completed, the pods are either retained or deleted based on the job's configuration. Jobs are ideal for short-lived, one-time tasks.

#### Key Features:
- Ensures a specified number of successful completions.
- Can run multiple pods in parallel to complete the task faster.
- Useful for tasks like data processing, backups, and batch processing.

#### Example YAML:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  completions: 1
  parallelism: 1
  template:
    metadata:
      labels:
        app: my-job
    spec:
      containers:
      - name: my-container
        image: my-image
        command: ["my-command"]
      restartPolicy: OnFailure
```

### CronJob
A CronJob is a Kubernetes resource that creates Jobs on a scheduled basis, similar to cron jobs in Unix/Linux. CronJobs are used for recurring tasks that need to run at specific times or intervals.

#### Key Features:
- Schedules jobs using standard cron syntax.
- Ensures jobs are created and executed according to the defined schedule.
- Useful for periodic tasks like database backups, report generation, and cleanup operations.

#### Example YAML:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: my-cronjob
        spec:
          containers:
          - name: my-container
            image: my-image
            command: ["my-command"]
          restartPolicy: OnFailure
```

### Differences Between Job and CronJob:
- **Job**: Runs once or a specified number of times until successful completion. It does not handle scheduling.
- **CronJob**: Schedules and runs Jobs at specified times or intervals using cron syntax.

### Summary
- **Job**: Use for one-time or fixed-count tasks. It's not scheduled but runs immediately when created.
- **CronJob**: Use for recurring tasks that need to run on a schedule. It automatically creates Jobs at specified intervals.
Both Jobs and CronJobs are essential for batch processing in Kubernetes, allowing you to handle both immediate and scheduled tasks efficiently.

---
**DaemonSet** —  Deamonset feature lets you run a Kubernetes pod on all(or selected) cluster nodes that meet certain criteria. Every time a new node is added to a cluster, the pod is added to it, and when a node is removed from the cluster, the pod is removed. example - monitoring tool pod, When a node is added or removed from the cluster, the DaemonSet ensures that the associated monitoring tools are also added or cleanly removed, keeping your cluster neat and efficient.

DaemonSets in Kubernetes are typically used for running system-level or cluster-wide services and are generally considered stateless. However, whether a DaemonSet is stateful or stateless depends on the specific use case and implementation of the pods it manages.

### DaemonSet Characteristics:
1. **Stateless Nature**:
   - DaemonSets usually run pods that provide services such as monitoring, logging, and networking, which do not require persistent state. Examples include Fluentd, Logstash, Prometheus Node Exporter, and CNI plugins.
   - These services collect data, process it, and often send it elsewhere (e.g., to a central logging server), hence not requiring persistent storage(so stateless).

2. **Potential Stateful Use Cases**:
   - While DaemonSets are commonly stateless, they can also be used for stateful workloads if each pod needs to maintain some local state on each node. An example might be a local caching service where each node keeps its own cache.
   - If a DaemonSet is used in a stateful manner, it may utilize PersistentVolumes to store state persistently on each node.

### Example Use Cases:

#### Stateless DaemonSet:
- **Fluentd DaemonSet**:
  ```yaml
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: fluentd
  spec:
    selector:
      matchLabels:
        app: fluentd
    template:
      metadata:
        labels:
          app: fluentd
      spec:
        containers:
        - name: fluentd
          image: fluentd:latest
  ```

#### Stateful DaemonSet:
- **Local Caching Service**:
  ```yaml
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: local-cache
  spec:
    selector:
      matchLabels:
        app: local-cache
    template:
      metadata:
        labels:
          app: local-cache
      spec:
        containers:
        - name: local-cache
          image: cache-image:latest
          volumeMounts:
          - name: cache-storage
            mountPath: /cache
      volumes:
      - name: cache-storage
        hostPath:
          path: /mnt/cache
          type: DirectoryOrCreate
  ```

### Summary
- **DaemonSets** are typically considered and used for stateless operations, running services that do not require persistent state across nodes.
- They can be configured for stateful operations if needed, using PersistentVolumes or local storage on each node.
- The primary role of DaemonSets is to ensure that a copy of a specific pod runs on all (or selected) nodes, providing node-level services essential for the cluster's operation.
