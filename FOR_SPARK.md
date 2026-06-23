# Kubernetes for Spark Developers

> As a Spark developer deploying applications on Kubernetes, you don't need to learn Kubernetes at the depth of a Kubernetes administrator or DevOps engineer. This guide focuses on the essentials needed to deploy, run, and troubleshoot Spark jobs effectively.

---

## 1. Kubernetes Fundamentals

### Core Concepts
- **Cluster**: Your Kubernetes infrastructure (Control Plane + Worker Nodes)
- **Control Plane**: Manages the cluster (API Server, etcd, Scheduler, Controller Manager)
- **Worker Nodes**: Run your applications
- **Pods**: Smallest deployable unit (usually one container per pod)
- **Containers**: Containerized applications (Docker images)
- **Namespaces**: Virtual clusters for resource organization and isolation
- **Labels & Selectors**: Metadata for organizing and selecting resources

### Learning Goal
Understand where your Spark driver and executor pods run, and how the cluster manages them.

---

## 2. Kubernetes Objects Used by Spark

### Essential Resources
| Resource | Purpose in Spark |
|----------|-----------------|
| **Pod** | Runs a single driver or executor container |
| **Deployment** | Manages replica sets of pods (rarely used directly with Spark) |
| **Service** | Exposes driver pod to external access (e.g., Spark UI) |
| **ConfigMap** | Stores application configuration files |
| **Secret** | Stores sensitive data (DB credentials, API keys) |
| **PV / PVC** | Persistent storage for data/checkpoints |

### Common Usage Examples
```yaml
# Database credentials stored as Secret
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: <base64-encoded>
  password: <base64-encoded>

# Application config stored as ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: spark-config
data:
  application.conf: |
    spark.default.parallelism=200
```

### Learning Goal
Know how Spark applications interact with these resources and how to mount them into your driver/executor pods.

---

## 3. Essential kubectl Commands

### Pod & Namespace Management
```bash
# View resources
kubectl get pods                          # List all pods in default namespace
kubectl get pods -n <namespace>           # List pods in specific namespace
kubectl get namespaces                    # List all namespaces

# Inspect resources
kubectl describe pod <pod-name>           # Detailed pod information
kubectl logs <pod-name>                   # View pod logs
kubectl logs <pod-name> -f                # Stream logs in real-time
kubectl logs <pod-name> -p                # View logs from previous container

# Debug & troubleshoot
kubectl exec -it <pod-name> -- bash       # Access pod shell
kubectl exec -it <pod-name> -- /bin/sh    # Access pod shell (Alpine/minimal images)
kubectl get events -n <namespace>         # View cluster events

# Cleanup
kubectl delete pod <pod-name>             # Delete a pod
kubectl delete pods --all -n <namespace>  # Delete all pods in namespace
```

### Learning Goal
Debug failed Spark jobs independently by inspecting pod logs, events, and resource usage.

---

## 4. Spark on Kubernetes Architecture

### Deployment Model
1. **Driver Pod**: Submits the Spark application and coordinates executors
2. **Executor Pods**: Execute tasks in parallel
3. **Dynamic Allocation**: Executor pods scale up/down based on workload

### Configuration Example
```bash
spark-submit \
  --master k8s://https://<k8s-apiserver>:6443 \
  --deploy-mode cluster \
  --name my-spark-job \
  --conf spark.executor.instances=5 \
  --conf spark.executor.memory=4g \
  --conf spark.executor.cores=2 \
  --conf spark.kubernetes.container.image=myrepo/spark:latest \
  --conf spark.kubernetes.namespace=spark-jobs \
  s3://my-bucket/my-app.jar
```

### Key Configuration Options
| Option | Purpose |
|--------|---------|
| `spark.executor.instances` | Number of executor pods |
| `spark.executor.memory` | Memory per executor (e.g., 4g) |
| `spark.executor.cores` | CPU cores per executor |
| `spark.kubernetes.container.image` | Docker image with Spark and your app |
| `spark.kubernetes.namespace` | Kubernetes namespace for pods |
| `spark.dynamicAllocation.enabled` | Enable/disable autoscaling executors |

### Learning Goal
Tune Spark applications for optimal resource usage and job completion.

---

## 5. Troubleshooting Common Issues

### Pod Status Issues

| Status | Cause | Solution |
|--------|-------|----------|
| **Pending** | Insufficient resources or unbound PVC | Check node capacity; verify PVC availability |
| **CrashLoopBackOff** | Application error or missing dependencies | Check logs: `kubectl logs <pod>` |
| **ImagePullBackOff** | Invalid/inaccessible Docker image | Verify image name, tag, and registry access |
| **OOMKilled** | Pod exceeded memory limit | Increase memory: `--conf spark.executor.memory=8g` |
| **Unknown/Evicted** | Node ran out of resources | Scale up cluster or reduce executor count |

### Debugging Workflow
```bash
# 1. Check pod status and events
kubectl describe pod <pod-name>

# 2. View logs (driver or executor)
kubectl logs <pod-name>

# 3. Check resource usage
kubectl top pod <pod-name>

# 4. Access pod shell if running
kubectl exec -it <pod-name> -- bash
```

### Learning Goal
Independently diagnose and fix common deployment and execution issues.

---

## 6. Resource Management

### CPU & Memory Requests/Limits
Requests ensure pod placement; limits prevent over-consumption.

```yaml
resources:
  requests:
    memory: "4Gi"
    cpu: "2"
  limits:
    memory: "8Gi"
    cpu: "4"
```

### Best Practices
- **Requests**: Set to expected resource usage
- **Limits**: Set 1.5x–2x higher than requests to handle spikes
- **Driver Pod**: Typically smaller (2–4 CPU, 2–4 GB memory)
- **Executor Pods**: Based on your workload (4–8 CPU, 4–16 GB memory common)

### Spark Configuration for Resources
```bash
--conf spark.kubernetes.driver.request.cores=2
--conf spark.kubernetes.driver.limit.cores=4
--conf spark.kubernetes.driver.memoryOverhead=512m
--conf spark.kubernetes.executor.request.cores=4
--conf spark.kubernetes.executor.limit.cores=8
--conf spark.kubernetes.executor.memoryOverhead=1g
```

### Learning Goal
Avoid executor failures and optimize cluster utilization by properly sizing resources.

---

## 7. Helm for Spark (Optional but Recommended)

### Core Concepts
- **Chart**: Pre-packaged Kubernetes manifests (like a template)
- **Release**: Deployed instance of a chart
- **values.yaml**: Configuration file to customize chart deployment
- **Helm Commands**: `install`, `upgrade`, `rollback`, `uninstall`

### Example: Installing Spark on Kubernetes with Helm
```bash
# Add Spark Helm repository
helm repo add spark-operator https://kubeflow.github.io/spark-operator

# Install Spark Operator
helm install spark-operator spark-operator/spark-operator \
  --namespace spark-operator \
  --create-namespace

# Deploy a Spark application
helm install my-spark-job ./spark-chart \
  --values custom-values.yaml \
  --namespace spark-jobs
```

### Learning Goal
Use Helm to streamline Spark application deployments and manage multiple releases.

---

## Resources & Next Steps

- [Spark on Kubernetes Documentation](https://spark.apache.org/docs/latest/running-on-kubernetes.html)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)