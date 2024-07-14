
# Kubernetes with Kind

## 1. Kind Cluster

### Create Cluster
```sh
kind create cluster --config cluster.yaml
```

### Delete Cluster
```sh
kind delete cluster --name development-cluster
```

### cluster.yaml
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: development-cluster
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

## 2. Pod

### Create Pod
```sh
kubectl create -f nginx-pod.yaml
```

### Delete Pod
```sh
kubectl delete -f nginx-pod.yaml
```

### nginx-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx
      image: nginx
```

### Apply Pod Configuration
```sh
kubectl apply -f nginx-pod.yaml
```

### Common Errors
- **ImagePullBackOff**: Failed to pull the image.
- **CrashLoopBackOff**: Image pulled but failed to start the container.

## Pod Labels

### pod-1.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    dept: dev-team
    team: team-a
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - name: "nginx-gateway"
          containerPort: 80
```

### pod-2.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
    dept: dev-team
    team: team-b
spec:
  containers:
    - name: nginx
      image: nginx
```

### pod-3.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
  labels:
    dept: devops-team
    team: team-a
spec:
  containers:
    - name: nginx
      image: nginx
```

### Query Pods
```sh
kubectl get pod -l dept=dev-team,team=team-a
```

### Port Forward
```sh
kubectl port-forward pod-1 8080:80
```

## Restart Policy

There are multiple `restartPolicy` options: `Always`, `Never`, `OnFailure`.

### pod-restart.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    dept: dev-team
    team: team-a
spec:
  restartPolicy: Always
  containers:
    - name: nginx
      image: nginx
      ports:
        - name: "nginx-gateway"
          containerPort: 80
```

## Entry Point in Kubernetes

Kubernetes has `command` and `args` just like Docker.

### busy-box-args.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busy-box
  labels:
    dept: dept-busy-box
spec:
  containers:
    - name: busy-box
      image: busybox
      args:
        - "ls"
        - "echo Hello World"
```

### busy-box-command.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busy-box
  labels:
    dept: dept-busy-box
spec:
  containers:
    - name: busy-box
      image: busybox
      command:
        - ls
```

## Kubernetes Logs

### View Logs
```sh
kubectl logs busy-box
```

## Terminate Grace Period

### busy-box-terminate.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busy-box
  labels:
    dept: dept-busy-box
spec:
  restartPolicy: Never
  terminationGracePeriodSeconds: 5
  containers:
    - name: busy-box
      image: busybox
      command: [ "sleep" ]
      args: [ "3600" ]
```

## Command vs Args

1. **Command** is used to override the entry point in a Docker image.
2. **Args** is used to provide arguments to the Docker entry point.

### busy-box-full.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busy-box
  labels:
    dept: dept-busy-box
spec:
  restartPolicy: Never
  terminationGracePeriodSeconds: 5
  containers:
    - name: busy-box
      image: busybox
      command: [ "sleep" ]
      args: [ "3600" ]
```

## Setup Environment Values

### busy-box-env.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busy-box
  labels:
    dept: dept-busy-box
spec:
  restartPolicy: Never
  terminationGracePeriodSeconds: 5
  containers:
    - name: busy-box
      image: busybox
      command: [ "sleep" ]
      args: [ "3600" ]
      env:
        - name: "spring.application.name"
          value: "WSD-Team"
```

## Explore Pod

### Exec into Pod
```sh
kubectl exec -it pod-1 -- bash
```

## Multi-Container Pod

### multi-container-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  restartPolicy: Never
  terminationGracePeriodSeconds: 5
  containers:
    - name: busybox-sleep
      image: busybox
      command: [ "sleep" ]
      args: [ "3600" ]
      env:
        - name: "spring.application.name"
          value: "WSD-Team"
    - name: busybox-echo
      image: busybox
      command: [ "sh", "-c" ]
      args: [ "while true; do echo Hello Kubernetes!; sleep 30; done" ]
      env:
        - name: "spring.application.name"
          value: "WSD-Team"
```

### Exec into Specific Container in Pod
```sh
kubectl exec -it multi-container-pod -c busybox-sleep -- sh
```

### Communication Between Containers in Same Pod
Containers in the same Pod can communicate with each other via `localhost`.

### Example:
```sh
curl localhost:8080
```

This README provides a comprehensive guide on setting up and managing Kubernetes Pods using Kind and kubectl, with examples of different configurations and commands.
