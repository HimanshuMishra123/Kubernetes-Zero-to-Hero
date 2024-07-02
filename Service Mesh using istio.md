Notes on Service Mesh using Istio with mutual TLS(mTLS) and a Canary Demo.This guide also covers admission controllers, sidecar containers, traffic management, mutual TLS (mTLS), and deployment strategies like canary deployments.

![image](https://github.com/HimanshuMishra123/Kubernetes-Zero-to-Hero/assets/164254902/2e688846-9967-4706-b0b1-9109497d9124)

## Service Mesh with Istio: Detailed Notes

### Key Concepts

#### What is a Service Mesh?
A service mesh helps manage the traffic of your Kubernetes cluster, particularly the east-west traffic (internal service-to-service communication). It enhances service-to-service communication by adding capabilities such as mutual TLS (mTLS), advanced deployment strategies, and observability.

#### East-West and North-South Traffic
- **East-West Traffic:** Internal traffic between services within the Kubernetes cluster.
- **North-South Traffic:** Traffic entering or exiting the Kubernetes cluster.

### Why Use a Service Mesh?

1. **Mutual TLS (mTLS):** Secures service-to-service communication by ensuring both parties in a communication exchange certificates and trust each other before communication is established.
2. **Advanced Deployment Strategies:** Simplifies the implementation of deployment strategies like canary, A/B testing, and blue-green deployments.
3. **Observability:** Provides out-of-the-box observability features, enabling tracking and monitoring of service behavior and metrics.
4. **Traffic Management:** Facilitates traffic splitting, circuit breaking, and other traffic management features.

### Installing and Configuring Istio

1. **Install Istio CLI:**
   ```sh
   curl -L https://istio.io/downloadIstio | sh -
   cd istio-*
   export PATH=$PWD/bin:$PATH
   ```

2. **Install Istio on Kubernetes:**
   ```sh
   istioctl install --set profile=demo -y
   ```

3. **Label Namespace for Istio Injection:**
   ```sh
   kubectl label namespace <namespace> istio-injection=enabled
   ```

### Istio Architecture Components

- **Istiod:** The control plane component managing configuration and lifecycle of the proxies.
- **Envoy Proxy(a proxy server):** A sidecar container injected into each pod which is a proxy server for handling all traffic in(ingress) and out(Egress) of the pod. As 

  ![Service Mesh explained in 60 minutes _ Istio mTLS and Canary Demo _ Complete beginner level guide 18-50 screenshot](https://github.com/HimanshuMishra123/Kubernetes-Zero-to-Hero/assets/164254902/61ded451-da23-444b-af67-43bc1152dc11)


### Istio Features

1. **Mutual TLS (mTLS):**
   - Secures communication by requiring both services to present valid certificates.
   - Example configuration for enabling mTLS:
     ```yaml
     apiVersion: security.istio.io/v1beta1
     kind: PeerAuthentication
     metadata:
       name: default
       namespace: <namespace>
     spec:
       mtls:
         mode: STRICT
     ```

2. **Traffic Management:**
   - **Virtual Services:** Define rules for routing traffic to different versions of a service.
   - **Destination Rules:** Defining "DestinationRule" in Istio is like setting rules for how your traffic should behave when it reaches a specific service in your network. (e.g. subset,load balancing, connection pool settings and outlier detection)

   - Example: Virtual service configuration for canary deployment:
     ```yaml
     apiVersion: networking.istio.io/v1beta1
     kind: VirtualService
     metadata:
       name: my-service
     spec:
       hosts:
       - my-service
       http:
       - route:
         - destination:
             host: my-service
             subset: v1
           weight: 90
         - destination:
             host: my-service
             subset: v2
           weight: 10
     ```
   - Example: Destination rule configuration 
      ```yaml
      apiVersion: networking.istio.io/v1beta1
      kind: DestinationRule
      metadata:
      name: my-destination-rule
      spec:
      host: my-service
      subsets:
      - name: v1
         labels:
            version: v1
      - name: v2
         labels:
            version: v2
      trafficPolicy:
         loadBalancer:
            simple: ROUND_ROBIN
         connectionPool:
            tcp:
            maxConnections: 100
            http:
            http1MaxPendingRequests: 100
            http2MaxRequests: 1000
         outlierDetection:
            consecutive5xxErrors: 2
            interval: 1s
            baseEjectionTime: 30s
      ```

you can create different DestinationRules for different versions of a service as well. This can be useful for managing specific settings and policies tailored to each version. example - different load balancing strategy needed for each version.  <br/>
![image](https://github.com/HimanshuMishra123/Kubernetes-Zero-to-Hero/assets/164254902/f5600d65-64db-4846-a2f7-1cbb06051c4f)


### Demo Application

We will use a multi-microservice application provided by Istio to demonstrate its features.

1. **Deploy the Demo Application:**
   ```sh
   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
   ```

2. **Accessing the Application:**
   ```sh
   kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
   ```

### Advanced Deployment Strategies

1. **Canary Deployment:**
   - Gradually shifts traffic to a new version of a service while monitoring its performance.
   - Adjust traffic weights based on performance metrics.

2. **A/B Testing:**
   - Route different subsets of users to different versions of a service for comparison.

3. **Blue-Green Deployment:**
   - Switch traffic between two environments (blue and green) with minimal downtime.

### Observability

- **Kiali:** Provides an observability dashboard for visualizing service mesh topology and metrics.
- **Jaeger:** Distributed tracing for monitoring request flows within the service mesh.
- **Prometheus:** Metrics collection and alerting.

### Configuring Admission Controllers

Istio uses dynamic admission controllers to manage sidecar injection and configuration changes.

1. **Verify Admission Controllers:**
   ```sh
   kubectl get pod -n kube-system -l component=kube-apiserver -o jsonpath='{.items[*].spec.containers[*].args}' | grep enable-admission-plugins
   ```

2. **Example of Mutation Admission Controller:**
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: my-claim
   spec:
     accessModes:
     - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```

   ```sh
   kubectl apply -f my-claim.yaml
   kubectl get pvc my-claim -o yaml
   ```

3. **Example of Validation Admission Controller:**
   ```yaml
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: mem-cpu-quota
     namespace: default
   spec:
     hard:
       requests.cpu: "1"
       requests.memory: "2Gi"
   ```

   ```sh
   kubectl apply -f resource-quota.yaml
   kubectl run test-pod --image=nginx --requests='cpu=500m,memory=1Gi'
   kubectl get pod test-pod -o yaml
   ```

### Conclusion

This guide covers the fundamental concepts and practical steps for implementing a service mesh with Istio, focusing on traffic management, mTLS, canary deployments, and observability. By following these steps, you can enhance the security, reliability, and observability of your Kubernetes cluster.

For more detailed commands and configurations, please refer to the [Istio documentation](https://istio.io/latest/docs/).

---
