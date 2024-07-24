# Securing K8s Cluster

## Securing K8s Cluster Part 1: Default Deny

In this guide, we'll explore how to enhance the security of a Kubernetes (K8s) cluster by implementing a "default deny" policy for pod-to-pod communication. This approach helps to minimize the attack surface by ensuring that only explicitly allowed traffic can flow between pods. We will demonstrate this process using two pods, `frontend` and `backend`, both running NGINX, and then apply a NetworkPolicy to restrict communication.

### Prerequisites
- A running Kubernetes cluster.
- Access to the Kubernetes control plane (kubectl installed and configured).

### Step 1: Create the `frontend` and `backend` Pods

First, we create two pods, `frontend` and `backend`, both using the NGINX image. We'll use imperative commands to simplify the process.

```bash
kubectl run frontend --image=nginx --restart=Never --labels="app=web,role=frontend"
kubectl run backend --image=nginx --restart=Never --labels="app=web,role=backend"
```

**Explanation:**
- `kubectl run`: This command creates and runs a new pod.
- `--image=nginx`: Specifies the container image to use, in this case, NGINX.
- `--restart=Never`: Ensures the pod will not be restarted automatically, aligning it with pod behavior rather than a Deployment or ReplicaSet.
- `--labels="app=web,role=frontend"`: Labels help identify and manage the pod; here, `app=web` and `role=frontend` distinguish the `frontend` pod.

### Step 2: Verify Connectivity Between Pods

Next, we check that the `frontend` and `backend` pods can communicate with each other before applying any NetworkPolicy.

1. Get the IP addresses of the pods:

   ```bash
   kubectl get pods -o wide
   ```

   Note the IP addresses of both `frontend` and `backend`.

2. Use the `kubectl exec` command to access the `frontend` pod and `curl` the `backend` pod:

   ```bash
   kubectl exec frontend -- curl <backend-pod-ip>
   ```

   Replace `<backend-pod-ip>` with the IP address of the `backend` pod.

   If the command returns the NGINX welcome page HTML, the connectivity is confirmed.

### Step 3: Apply a NetworkPolicy to Deny Ingress Traffic

We now apply a NetworkPolicy that denies all ingress traffic to pods in the default namespace unless explicitly allowed.

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
```

Save the above YAML to a file named `deny-all-ingress.yaml`, and apply it using:

```bash
kubectl apply -f deny-all-ingress.yaml
```

**Explanation:**
- `podSelector: {}`: This selects all pods in the namespace. Since no specific labels are provided, it applies to all pods.
- `policyTypes: - Ingress`: Specifies that this policy controls ingress traffic.

### Step 4: Verify the NetworkPolicy

To confirm that the NetworkPolicy is effective, repeat the connectivity test:

```bash
kubectl exec frontend -- curl <backend-pod-ip>
```

This time, the command should timeout or fail, indicating that the `frontend` pod cannot access the `backend` pod due to the applied NetworkPolicy.

### Conclusion

In this guide, we demonstrated how to secure a Kubernetes cluster by implementing a "default deny" policy for ingress traffic using NetworkPolicy. This is a fundamental step in Kubernetes security, as it restricts pod-to-pod communication, helping to minimize potential attack vectors.

In the next part of this series, we will explore how to refine these policies to allow necessary traffic while maintaining security.
