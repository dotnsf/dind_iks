# Docker in Docker


## How to use(Cluster internal use)

```dind.yml

$ kubectl apply -f https://raw.githubusercontent.com/dotnsf/dind_iks/main/dind.yml

$ kubectl exec -it dind -c docker -- /bin/sh

/ # docker run -d --name nginx -p 8000:80 nginx

/ # apk add --update curl w3m

/ # w3m http://localhost:8000/

/ # exit
```


## How to use(Expose port 8000 as 30800 to internet)

```dind_expose.yml

$ kubectl apply -f https://raw.githubusercontent.com/dotnsf/dind_iks/main/dind_expose.yml

$ kubectl get all

NAME                       READY   STATUS    RESTARTS   AGE
pod/dind-xxxxxxxxx-xxxxx   2/2     Running   1          19s
  :
  :

$ kubectl exec -it dind-xxxxxxxxx-xxxxx -c docker -- /bin/sh

/ # docker run -d --name nginx -p 8000:80 nginx

(access to http://(IP of worker node):30800/ from outside of cluster )
```


## YAML files

```dind.yml

apiVersion: v1
kind: Pod
metadata:
  name: dind
spec:
  containers:
    - name: docker
      image: docker:19.03
      command: ["docker", "run", "nginx:latest"]
      env:
        - name: DOCKER_HOST
          value: tcp://localhost:2375
    - name: dind-daemon
      image: docker:19.03-dind
      env:
        - name: DOCKER_TLS_CERTDIR
          value: ""
      resources:
        requests:
          cpu: 20m
          memory: 512Mi
      securityContext:
        privileged: true

```

```dind_expose.yml

apiVersion: v1
kind: Service
metadata:
  name: dind
spec:
  selector:
    app: dind
  ports:
  - port: 8000
    name: port8000
    protocol: TCP
    targetPort: 8000
    nodePort: 30800
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dind
spec:
  selector:
    matchLabels:
      app: dind
  replicas: 1
  template:
    metadata:
      labels:
        app: dind
    spec:
      containers:
        - name: docker
          image: docker:19.03
          command: ["docker", "run", "nginx:latest"]
          env:
            - name: DOCKER_HOST
              value: tcp://localhost:2375
        - name: dind-daemon
          image: docker:19.03-dind
          env:
            - name: DOCKER_TLS_CERTDIR
              value: ""
          resources:
            requests:
              cpu: 20m
              memory: 512Mi
          securityContext:
            privileged: true

```


## Licensing

This code is licensed under MIT.


## Copyright

2022  [K.Kimura @ Juge.Me](https://github.com/dotnsf) all rights reserved.
