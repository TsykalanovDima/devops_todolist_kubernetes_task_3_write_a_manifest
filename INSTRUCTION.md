# Kubernetes Deployment Instructions for ToDo App

## Prerequisites

- A running Kubernetes cluster (e.g., Minikube, kind, or a cloud provider)
- `kubectl` installed and configured to communicate with your cluster
- Docker installed (for building/pushing the image)

---

## 1. Build and Push the Docker Image

```bash
docker build -t dtsy/todoapp:3.0.0 .
docker push dtsy/todoapp:3.0.0
```

---

## 2. Apply All Manifests

Apply the manifests in the following order:

### Step 1 — Create the Namespace

```bash
kubectl apply -f .infrastructure/namespace.yml
```

Verify:

```bash
kubectl get namespaces
```

### Step 2 — Deploy the BusyBox Pod

```bash
kubectl apply -f .infrastructure/busybox.yml
```

### Step 3 — Deploy the ToDo App Pod

```bash
kubectl apply -f .infrastructure/todoapp-pod.yml
```

Verify all pods are running:

```bash
kubectl get pods -n todoapp
```

Wait until the ToDo app pod shows `Running` and `READY 1/1` — this confirms the readiness probe passed.

---

## 3. Test the ToDo Application via `port-forward`

```bash
kubectl port-forward pod/todoapp -n todoapp 8000:8000
```

In a separate terminal:

```bash
# Landing page
curl http://localhost:8000/

# API root
curl http://localhost:8000/api/

# Readiness probe endpoint
curl http://localhost:8000/api/readyz/

# Liveness probe endpoint
curl http://localhost:8000/api/healthz/
```

Expected responses:
- `/api/readyz/` → `ready` (HTTP 200)
- `/api/healthz/` → `alive` (HTTP 200)

Press `Ctrl+C` to stop the port-forward.

---

## 4. Test the Application Using the `busyboxplus:curl` Container

Get the ToDo app pod's cluster IP:

```bash
kubectl get pod todoapp -n todoapp -o wide
```

Exec into the BusyBox pod:

```bash
kubectl exec -it busybox -n todoapp -- sh
```

From inside the container (replace `<TODOAPP_POD_IP>` with the IP from above):

```bash
curl http://<TODOAPP_POD_IP>:8000/api/readyz/
# Expected: ready

curl http://<TODOAPP_POD_IP>:8000/api/healthz/
# Expected: alive

curl http://<TODOAPP_POD_IP>:8000/api/
# Expected: API root response

curl http://<TODOAPP_POD_IP>:8000/
# Expected: landing page HTML
```

---

## 5. Cleanup

```bash
kubectl delete -f .infrastructure/todoapp-pod.yml
kubectl delete -f .infrastructure/busybox.yml
kubectl delete -f .infrastructure/namespace.yml
```