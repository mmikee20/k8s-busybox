Deploy and Operate a Public Container (BusyBox) in Kubernetes
===================================================================

Description
============
Goal
Deploy a busybox container to an existing Kubernetes namespace. The container will run in an infinite loop, printing a configurable message every few seconds. This task focuses on learning resource definitions, configuration injection, container lifecycle management, and basic Kubernetes operations.

Learning Objectives
=====================
Deploy a public container using a Deployment

Use a ConfigMap to inject environment variables

Inspect logs, environment variables, and running processes inside a Pod

Update a running deployment using kubectl

Practice safe deletion and resource cleanup

1. Configuration and Deployment
=========================================
Create the following manifests in a folder called k8s-busybox/.

configmap.yaml
========================
apiVersion: v1
kind: ConfigMap
metadata:
  name: busybox-config
data:
  MESSAGE: "Hello from BusyBox in Kubernetes"


deployment.yaml
========================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
        - name: busybox
          image: busybox:1.36
          command: ["/bin/sh", "-c"]
          args:
            - while true; do
                echo "$(date) - $MESSAGE";
                sleep 5;
              done;
          env:
            - name: MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: busybox-config
                  key: MESSAGE


2. Apply Resources
========================
kubectl apply -f k8s-busybox/configmap.yaml
kubectl apply -f k8s-busybox/deployment.yaml


3. Inspect the Deployment
========================
Check pod status:

kubectl get pods
View logs:

kubectl logs -l app=busybox
Exec into the container:

kubectl exec -it deploy/busybox-deployment -- sh
View injected env variables:

kubectl exec -it deploy/busybox-deployment -- printenv


4. Test Updates
========================
Modify MESSAGE in the ConfigMap:

MESSAGE: "Updated message from BusyBox"
Apply the updated ConfigMap:

kubectl apply -f k8s-busybox/configmap.yaml
Restart the pod to reload the config:

kubectl rollout restart deployment busybox-deployment
Verify the new log output reflects the change.


5. Cleanup (Optional)
========================
kubectl delete -f k8s-busybox/


Stretch Goals
========================
Add a liveness probe to check if the process is still running
Add resource requests and limits
Convert the workload into a CronJob that runs once every minute
Try running the container with a persistent volume mount (e.g., emptyDir)


Learning material
========================
Kubernetes Deployment: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ 
Kubernetes ConfigMap: https://kubernetes.io/docs/concepts/configuration/configmap/ 
BusyBox image: https://hub.docker.com/_/busybox
