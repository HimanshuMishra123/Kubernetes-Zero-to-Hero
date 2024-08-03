### StatefulSet vs. DaemonSet vs. Deployments vs. ReplicaSet

![Difference-between-ReplicaSet-ReplicationController-2](https://github.com/user-attachments/assets/977d2848-22bf-414f-833f-f15422cf0eb7)

![statefulSets-in-kubernetes-ezgif com-webp-to-jpg-converter](https://github.com/user-attachments/assets/15d7a0d6-4a91-480f-8986-4316990d89b9)


How stateful apps are different than stateless apps? As we will see, the `state` aspect of stateful apps makes orchestration more complex than what the initial Kubernetes controllers were built for.

**Stateful Applications** are applications that are mindful of their past and present state. In a nutshell, the applications that keep track of their state. They store data using persistent storage and read the data later to survive service breakdown or restarts. Database applications like MySQL and MongoDB are examples of stateful applications. StatefulSet is the controller that manages the deployment and scaling of a set of Stateful application pods. A stateful pod in Kubernetes is a pod that requires persistent storage and a stable network identity to maintain its state all the time, even during pod restarts or rescheduling. Unlike a Deployment, which manages stateless applications, a StatefulSet maintains a unique identity for each of its pods and ensures that they are deployed in a predictable order.

On the other hand, **Stateless Applications** are applications that do not monitor any of their states. They neither store nor read data from any storage; they are basically a one-time request and feedback process. When a stateless application’s current session is down, interrupted or deleted, the new session will start with a clean slate, without referring to past events or processes. Examples include the Nginx web application and Tomcat web server.

### Voulme attachment
![c0e51-1jcqz09xnlqufsm3sl-ixvw](https://github.com/user-attachments/assets/8b473244-fd2f-46b8-8085-c7bf6d760556)

As highlighted in Containerizing Stateful Applications, data volume mapped to a container can be either:

1. Inside the container

2. Inside the host, independent of the container

3. Outside the host, independent of both the host and the container

Stateless applications by definition have no need for long-running persistence and hence usually rely on the first 2 options. However, stateful applications (such as databases, message queues, metadata stores) have to use the 3rd approach in order to guarantee that the data outlives the lifecycle of a container or a host. The above also means that as new containers or new hosts are brought up, we should be able to map them to existing persistent volumes.

#### Key Features of Statefulset:
![20190808_162532179_90804 (1)](https://github.com/user-attachments/assets/36579d94-b60f-4af9-9d13-25e232db263f)

- **Stable, unique network identifiers:** Each pod in a StatefulSet has a unique, stable network identity that persists across rescheduling.
- **Stable storage:** StatefulSets can provide stable storage using PersistentVolumes, ensuring that each pod has a persistent disk even if it is rescheduled.
- **Ordered, graceful deployment and scaling:** Pods are created in sequential order, and scaling operations are performed in a controlled manner.
- **Ordered, automated rolling updates:** Updates to the pods in a StatefulSet are done in a specific, ordered manner to ensure minimal disruption.

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
- **PersistentVolumeClaims (PVCs):** Users request PVs by creating PersistentVolumeClaims (PVCs). The cluster then binds the PVC to a suitable PV, ensuring the user gets the requested storage.

#### Common Use Cases PV:
- Storing application data
- Maintaining logs
- Providing shared storage for stateful applications

Read  example of reddis as statefulset with PV from... 
https://github.com/HimanshuMishra123/three-tier-architecture-demo/blob/master/self-readme.md

### DaemonSet vs. Deployments vs. ReplicaSet
![Difference-between-ReplicaSet-ReplicationController-2](https://github.com/user-attachments/assets/977d2848-22bf-414f-833f-f15422cf0eb7)
![6eac4-1fhhtm36vopagnytud1wdjq](https://github.com/user-attachments/assets/e6946205-af4d-4edb-be78-5b2e839003d6)
Common Kubernetes Controller APIs Not Fit For Stateful Apps

ReplicaSet — used to run N identical pods but with ephemeral storage only. No support for persistent storage so clearly not a fit for stateful apps.

Deployment — extends the ReplicaSet functionality to add update semantics that allows pod recreation and rolling upgrades. De-facto standard for stateless apps.

DaemonSet —  Deamonset feature lets you run a Kubernetes pod on all cluster nodes that meet certain criteria. Every time a new node is added to a cluster, the pod is added to it, and when a node is removed from the cluster, the pod is removed. example - monitoring tool pod, When a node is added or removed from the cluster, the DaemonSet ensures that the associated monitoring tools are also added or cleanly removed, keeping your cluster neat and efficient.

Job — used for running batch jobs at a pre-determined schedule.

As we can see, none of the above controllers are suitable for running stateful apps. Hence, the need for StatefulSets, a special controller purpose-built for the needs of stateful apps.
