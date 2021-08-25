# Center-wide Parallel Filesystems

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    ccs.ornl.gov/fs: olcf
  labels:
    app: minio-gateway
  name: minio-gateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minio-gateway
  template:
    metadata:
      labels:
        app: minio-gateway
    spec:
      containers:
      - args:
        - gateway
        - nas
        - /ccs/proj/stf042
        - --console-address
        - :9001
        image: minio/minio
        name: minio
        env:
        - name: MINIO_ROOT_USER
          value: accesskey
        - name: MINIO_ROOT_PASSWORD
          value: secretkey
        - name: MINIO_SERVER_URL
          value: http://minio-gateway:9000
        - name: MINIO_BROWSER_REDIRECT_URL
          value: https://minio-console-stf042.apps.marble.ccs.ornl.gov
        ports:
        - containerPort: 9000
          name: minio
        - containerPort: 9001
          name: console
        resources: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: minio-gateway
  name: minio-gateway
spec:
  ports:
  - name: minio
    port: 9000
    protocol: TCP
    targetPort: 9000
  - name: console
    port: 9001
    protocol: TCP
    targetPort: 9001
  selector:
    app: minio-gateway
  type: ClusterIP
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: minio-gateway
  name: minio-console
spec:
  host: minio-console-stf042.apps.marble.ccs.ornl.gov
  port:
    targetPort: console
  tls:
    insecureEdgeTerminationPolicy: Redirect
    termination: edge
  to:
    kind: Service
    name: minio-gateway
    weight: 100
```
