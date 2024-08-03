### StatefulSet vs. DaemonSet vs. Deployments vs. ReplicaSet

![Difference-between-ReplicaSet-ReplicationController-2](https://github.com/user-attachments/assets/977d2848-22bf-414f-833f-f15422cf0eb7)
![6eac4-1fhhtm36vopagnytud1wdjq](https://github.com/user-attachments/assets/d126fc1f-afb7-4568-918a-a6dc0e462406)




How stateful apps are different than stateless apps? As we will see, the `state` aspect of stateful apps makes orchestration more complex than what the initial Kubernetes controllers were built for.

Stateful Applications are applications that are mindful of their past and present state. In a nutshell, the applications that monitor or keep track of their state. They store data using persistent storage and read the data later to survive service breakdown or restarts. Database applications like MySQL and MongoDB are examples of stateful applications.

On the other hand, Stateless Applications are applications that do not monitor any of their states. They neither store nor read data from any storage; they are basically a one-time request and feedback process. When a stateless application’s current session is down, interrupted or deleted, the new session will start with a clean slate, without referring to past events or processes. Examples include the Nginx web application and Tomcat web server.
