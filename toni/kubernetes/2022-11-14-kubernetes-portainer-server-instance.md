# Deployment Portainer on Kubernetes

source: 
- [https://docs.portainer.io/start/install/server/kubernetes/baremetal](https://docs.portainer.io/start/install/server/kubernetes/baremetal)

## Deployment

deployment portainer dalam kubernetes cluster bisa menggunakan helm charts or YAML manifests.

### Deploy via Helm

```
helm repo add portainer https://portainer.github.io/k8s/
help repo update
```

ada 3 cara install utk meng-expose service portainer:

- via load balancer ip:9443.

```
helm install --create-namespace -n portainer portainer portainer/portainer \
    --set service.type=LoadBalancer \
    --set tls.force=true
```

> hapus `--set tls.force=true` utk mengakses portainer via port 9000 HTTP

> portainer menggunakan self-signed ssl utk secure port 9443. atau menggunakan ssl kita saat instalasi/via portainer ui setelah install

- via Ingress

portainer akan di deploy ke cluster dan menggunakan assigned Cluster IP, dg nginx ingress controller pada hostname tertentu.

```
helm install --create-namespace -n portainer portainer portainer/portainer \
    --set service.type=ClusterIP \
    --set tls.force=true \
    --set ingress.enabled=true \
    --set ingress.ingressClassName=<ingressClassName (eg: nginx)> \
    --set ingress.annotations."nginx\.ingress\.kubernetes\.io/backend-protocol"=HTTPS \
    --set ingress.hosts[0].host=<fqdn (eg: portainer.example.io)> \
    --set ingress.hosts[0].paths[0].path="/"
```

- via NodePort

secara default portainer di deploy forward pada ports `30779` HTTPS

hapus `--set tls.force=true` utk mengakses portainer via HTTP di port `30777`

```
helm install --create-namespace -n portainer portainer portainer/portainer --set tls.force=true
```

> utk memilih spesifik node target utk install portainer via CLI, incluse `--set nodeSelector.kubernetes.io/hostname=<NODE-NAME>` di `helm install`

### Deploy via YAML manifests

- via Load Balancer

Portainer provision an assigned Load Balancer IP on port `9000` HTTP dan `9443` HTTPS

```
kubectl apply -n portainer -f https://downloads.portainer.io/ce2-16/portainer-lb.yaml
```

- via NodePort

Portainer available port `30777` HTTP dan `30779` HTTPS

```
kubectl apply -n portainer -f https://downloads.portainer.io/ce2-16/portainer.yaml
```

> pilih node spesifik saat deploy YAML manifests, one-liner patch deployment, force pod always scheduled on node its currently running:

```
kubectl patch deployments -n portainer portainer -p '{"spec": {"template": {"spec": {"nodeSelector": {"kubernetes.io/hostname": "'$(kubectl get pods -n portainer -o jsonpath='{ ..nodeName }')'"}}}}}' || (echo Failed to identify current node of portainer pod; exit 1)
```

[Appendix YAML](#appendix-yaml)

## Log In

- via Load Balancer

```
# gunakan IP address dan port / FQDN dari loadbalancer
https://<loadbalancer IP>:9443/ 
# or 
http://<loadbalancer IP>:9000/
```

- via Ingress

```
https://<FQDN>/
# fully-qualified domain-name, contoh k8s.local.lan or k8s.example.com
```

- via NodePort

```
# ganti `localhost` dg relevant IP / FQDN jika perlu, dan adjust port jika berbeda
https://localhost:30779/
# or 
http://localhost:30777/
```

### Appendix

- [https://downloads.portainer.io/ce2-16/portainer.yaml](https://downloads.portainer.io/ce2-16/portainer.yaml)

```yaml
---
# Source: portainer/templates/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: portainer
---
# Source: portainer/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: portainer-sa-clusteradmin
  namespace: portainer
  labels:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
---
# Source: portainer/templates/pvc.yaml
kind: "PersistentVolumeClaim"
apiVersion: "v1"
metadata:
  name: portainer
  namespace: portainer  
  annotations:
    volume.alpha.kubernetes.io/storage-class: "generic"
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
spec:
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: "10Gi"
---
# Source: portainer/templates/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: portainer
  labels:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  namespace: portainer
  name: portainer-sa-clusteradmin
---
# Source: portainer/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: portainer
  namespace: portainer
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
spec:
  type: NodePort
  ports:
    - port: 9000
      targetPort: 9000
      protocol: TCP
      name: http
      nodePort: 30777
    - port: 9443
      targetPort: 9443
      protocol: TCP
      name: https
      nodePort: 30779      
    - port: 30776
      targetPort: 30776
      protocol: TCP
      name: edge
      nodePort: 30776
  selector:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
---
# Source: portainer/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: portainer
  namespace: portainer
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
spec:
  replicas: 1
  strategy:
    type: "Recreate"
  selector:
    matchLabels:
      app.kubernetes.io/name: portainer
      app.kubernetes.io/instance: portainer
  template:
    metadata:
      labels:
        app.kubernetes.io/name: portainer
        app.kubernetes.io/instance: portainer
    spec:
      nodeSelector:
        {}
      serviceAccountName: portainer-sa-clusteradmin
      volumes:
        - name: "data"
          persistentVolumeClaim:
            claimName: portainer
      containers:
        - name: portainer
          image: "portainer/portainer-ce:2.16.1"
          imagePullPolicy: Always
          args:
          - '--tunnel-port=30776'          
          volumeMounts:
            - name: data
              mountPath: /data              
          ports:
            - name: http
              containerPort: 9000
              protocol: TCP
            - name: https
              containerPort: 9443
              protocol: TCP              
            - name: tcp-edge
              containerPort: 8000
              protocol: TCP              
          livenessProbe:
            httpGet:
              path: /
              port: 9443
              scheme: HTTPS
          readinessProbe:
            httpGet:
              path: /
              port: 9443
              scheme: HTTPS
          resources:
            {}
```


- [https://downloads.portainer.io/ce2-16/portainer-lb.yaml](https://downloads.portainer.io/ce2-16/portainer-lb.yaml)

```yaml
---
# Source: portainer/templates/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: portainer
---
# Source: portainer/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: portainer-sa-clusteradmin
  namespace: portainer
  labels:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
---
# Source: portainer/templates/pvc.yaml
kind: "PersistentVolumeClaim"
apiVersion: "v1"
metadata:
  name: portainer
  namespace: portainer  
  annotations:
    volume.alpha.kubernetes.io/storage-class: "generic"
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
spec:
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: "10Gi"
---
# Source: portainer/templates/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: portainer
  labels:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  namespace: portainer
  name: portainer-sa-clusteradmin
---
# Source: portainer/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: portainer
  namespace: portainer
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
spec:
  type: LoadBalancer
  ports:
    - port: 9000
      targetPort: 9000
      protocol: TCP
      name: http
    - port: 9443
      targetPort: 9443
      protocol: TCP
      name: https
    - port: 8000
      targetPort: 8000
      protocol: TCP
      name: edge
  selector:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
---
# Source: portainer/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: portainer
  namespace: portainer
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.1"
spec:
  replicas: 1
  strategy:
    type: "Recreate"
  selector:
    matchLabels:
      app.kubernetes.io/name: portainer
      app.kubernetes.io/instance: portainer
  template:
    metadata:
      labels:
        app.kubernetes.io/name: portainer
        app.kubernetes.io/instance: portainer
    spec:
      nodeSelector:
        {}
      serviceAccountName: portainer-sa-clusteradmin
      volumes:
        - name: "data"
          persistentVolumeClaim:
            claimName: portainer
      containers:
        - name: portainer
          image: "portainer/portainer-ce:2.16.1"
          imagePullPolicy: Always
          args:          
          volumeMounts:
            - name: data
              mountPath: /data              
          ports:
            - name: http
              containerPort: 9000
              protocol: TCP
            - name: https
              containerPort: 9443
              protocol: TCP              
            - name: tcp-edge
              containerPort: 8000
              protocol: TCP              
          livenessProbe:
            httpGet:
              path: /
              port: 9443
              scheme: HTTPS
          readinessProbe:
            httpGet:
              path: /
              port: 9443
              scheme: HTTPS
          resources:
            {}
```

