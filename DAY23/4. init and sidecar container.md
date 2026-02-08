# Kubernetes Init Containers & Sidecar Containers

---

## 1. Init Containers Example (Pre-Start Tasks)

**Purpose:**  
Init containers run before the main app container starts. They are useful for:
- Waiting for services
- Setting up configs
- Pre-start checks

---

### YAML: Init Container Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-demo
  namespace: test-ns
spec:
  containers:
  - name: main-app
    image: busybox
    command: ['sh', '-c', 'echo Main App Running; sleep 3600']
  initContainers:
  - name: init-wait
    image: busybox
    command: ['sh', '-c', 'echo Init Task Starting; sleep 10; echo Init Task Done']
````

---

### Commands to Apply & Check Logs:

```bash
kubectl apply -f init-container.yaml
kubectl get pods -n test-ns -w
```

**Check Init Container Logs:**

```bash
kubectl logs init-container-demo -n test-ns -c init-wait
```

**Check Main App Logs:**

```bash
kubectl logs init-container-demo -n test-ns -c main-app
```

---

### Explanation:

* Init container runs first, waits for 10 seconds, prints logs, then exits.
* Only after it finishes, the main app starts.

---

## 2. Sidecar Containers Example (Run Together)

**Purpose:**
Sidecar containers run together with the main app and are useful for:

* Logging agents
* Proxies
* Monitoring tools

---

### YAML: Sidecar Container Example (Log Writer + Log Watcher)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-container-demo
  namespace: test-ns
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}

  containers:
  - name: main-app
    image: busybox
    command: ['sh', '-c', 'while true; do echo $(date) Writing Logs >> /logs/app.log; sleep 5; done']
    volumeMounts:
    - name: shared-logs
      mountPath: /logs

  - name: sidecar-log-watcher
    image: busybox
    command: ['sh', '-c', 'tail -f /logs/app.log']
    volumeMounts:
    - name: shared-logs
      mountPath: /logs
```

---

### Commands to Apply & Check Logs:

```bash
kubectl apply -f sidecar-container.yaml
kubectl get pods -n test-ns -w
```

**Check Sidecar Container Logs (Tail Logs):**

```bash
kubectl logs sidecar-container-demo -n test-ns -c sidecar-log-watcher
```

**Check Main App Container Logs:**

```bash
kubectl logs sidecar-container-demo -n test-ns -c main-app
```

**To Check Logs Inside Container (Verify File Logs):**

```bash
kubectl exec -it sidecar-container-demo -n test-ns -c main-app -- sh
cat /logs/app.log
```

---

### Explanation:

* Both containers share `/logs` volume.
* Main app writes logs to file continuously.
* Sidecar container tails and prints those logs live.

---

## Key Differences Between Init & Sidecar Containers

| Feature             | Init Container                         | Sidecar Container                   |
| ------------------- | -------------------------------------- | ----------------------------------- |
| When Runs           | Before main container starts           | Together with main container        |
| Purpose             | Setup tasks (DB migration, pre-checks) | Helper tasks (logging, monitoring)  |
| Runs Together?      | No                                     | Yes                                 |
| Logs Check Method   | Logs disappear after completion        | Logs always available while running |
| Real-world Examples | DB schema setup, wait for DB           | Logging agents, service proxies     |

---

## Diagram (Visual Flow)

```
+-------------------+      +------------------------+
| Init Container(s) | ---> | Main + Sidecar Running |
+-------------------+      +------------------------+
        (Sequential)                (Parallel)
```

---

## Common Trainer Tips:

* Init Containers always run first and must finish successfully.
* Sidecar Containers run side by side with main app for additional functionality.
* Both concepts are common in Microservice & DevOps deployments.

---

## Useful Commands Cheat Sheet:

```bash
kubectl get pods -n test-ns -w

kubectl logs <pod-name> -n test-ns -c <container-name>

kubectl exec -it <pod-name> -n test-ns -c <container-name> -- sh
```

---

### How To Explain in Interviews:

* **Init containers** are like **pre-flight checks** before the app launches.
* **Sidecar containers** are like **helpers** that work alongside main apps.
* Both enhance **modularity** and **reusability** of Kubernetes apps.


