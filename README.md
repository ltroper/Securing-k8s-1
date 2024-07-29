# Securing K8s Cluster

## Securing K8s Cluster Part 1: Default Deny

In this guide, we'll explore how to enhance the security of a Kubernetes (K8s) cluster by implementing a "default deny" policy for pod-to-pod communication. This approach helps to minimize the attack surface by ensuring that only explicitly allowed traffic can flow between pods. We will demonstrate this process using two pods, `frontend` and `backend`, both running NGINX, and then apply a NetworkPolicy to restrict communication.

### Prerequisites
- A running Kubernetes cluster.
- Access to the Kubernetes control plane (kubectl installed and configured).

### Step 1: Create the `frontend` and `backend` Pods

First, we create two pods, `frontend` and `backend`, both using the NGINX image. We'll use imperative commands to simplify the process.

```bash
kubectl run frontend --image=nginx --labels="app=web,role=frontend"
kubectl run backend --image=nginx --labels="app=web,role=backend"
```

**Explanation:**
- `kubectl run`: This command creates and runs a new pod.
- `--image=nginx`: Specifies the container image to use, in this case, NGINX.
- `--labels="app=web,role=frontend"`: Labels help identify and manage pods.

### Step 2: Expose the Pods

To allow the pods to communicate with each other, we need to expose them. This can be done using Kubernetes Services:

```bash
kubectl expose pod frontend --port=80 
kubectl expose pod backend --port=80 
```

**Explanation:**
- `kubectl expose`: This command creates a Service object, which provides a stable IP address and DNS name for accessing the pod.
- `--port=80`: The port that the service exposes.



### Step 3: Verify Connectivity Between Pods

Next, we check that the `frontend` and `backend` pods can communicate with each other before applying any NetworkPolicy.


1. Use the `kubectl exec` command to access the `frontend` pod and `curl` the `backend` pod:

   ```bash
   kubectl exec frontend -- curl backend
   ```
   ```bash
   kubectl exec backend -- curl front
   ```

   
   If the commands return the NGINX welcome page HTML, the connectivity is confirmed.

### Step 4: Apply a NetworkPolicy to Deny Ingress and Egress Traffic

We now apply a NetworkPolicy that denies all traffic to and from pods in the default namespace unless explicitly allowed.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

Save the above YAML to a file named `deny-all.yaml`, and apply it using:

```bash
kubectl apply -f deny-all.yaml
```

**Explanation:**
- `podSelector: {}`: This selects all pods in the namespace. Since no specific labels are provided, it applies to all pods.

### Step 5: Verify the NetworkPolicy

To confirm that the NetworkPolicy is effective, repeat the connectivity test:

```bash
kubectl exec frontend -- curl backend
```

This time, the command should timeout or fail, indicating that the `frontend` pod cannot access the `backend` pod, and viceversa, due to the applied NetworkPolicy.

Sure, here are the additional sections to allow connectivity between the `frontend` and `backend` pods by defining specific NetworkPolicies.

### Step 6: Allow Egress Traffic from `frontend` to `backend`

Now, we will create a NetworkPolicy to allow egress traffic from the `frontend` pod to the `backend` pod. 

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-egress-to-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - protocol: TCP
      port: 80
```

Save the above YAML to a file named `allow-frontend-egress-to-backend.yaml`, and apply it using:

```bash
kubectl apply -f allow-frontend-egress-to-backend.yaml
```

**Explanation:**
- `podSelector: { matchLabels: { role: frontend } }`: This policy applies to pods with the label `role=frontend`.
- `policyTypes: - Egress`: Specifies that this policy controls egress traffic.
- `egress: - to: { podSelector: { matchLabels: { role: backend } } }`: Allows egress traffic to pods with the label `role=backend`.
- `ports: - protocol: TCP, port: 80`: Restricts the allowed egress traffic to TCP port 80.

### Step 7: Allow Ingress Traffic to `backend` from `frontend`

Next, we will create a NetworkPolicy to allow ingress traffic to the `backend` pod from the `frontend` pod.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-ingress-from-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
```

Save the above YAML to a file named `allow-backend-ingress-from-frontend.yaml`, and apply it using:

```bash
kubectl apply -f allow-backend-ingress-from-frontend.yaml
```

**Explanation:**
- `podSelector: { matchLabels: { role: backend } }`: This policy applies to pods with the label `role=backend`.
- `policyTypes: - Ingress`: Specifies that this policy controls ingress traffic.
- `ingress: - from: { podSelector: { matchLabels: { role: frontend } } }`: Allows ingress traffic from pods with the label `role=frontend`.
- `ports: - protocol: TCP, port: 80`: Restricts the allowed ingress traffic to TCP port 80.

### Step 8: Verify the Connectivity

After applying these policies, verify that the `frontend` pod can communicate with the `backend` pod:

```bash
kubectl exec frontend -- curl backend
```

This command should now succeed, indicating that the `frontend` pod can access the `backend` pod on port 80, while the default deny policy ensures that all other traffic is restricted.

### Conclusion

In this guide, we have demonstrated how to enhance the security of a Kubernetes cluster by implementing a "default deny" policy and then selectively allowing necessary traffic between specific pods using NetworkPolicies. This approach helps to minimize the attack surface by ensuring that only explicitly allowed traffic can flow between pods.

In the next part of this series, we will explore how to further refine these policies to allow necessary traffic while maintaining security and other ways of securing your cluster.









