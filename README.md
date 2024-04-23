# kubernetes-ckad

### Test - Labs Udemy

1. Create a Persistent Volume called log-volume. It should make use of a storage class name manual. It should use RWX as the access mode and have a size of 1Gi. The volume should use the hostPath /opt/volume/nginx
Next, create a PVC called log-claim requesting a minimum of 200Mi of storage. This PVC should bind to log-volume.
Mount this in a pod called logger at the location /var/www/nginx. This pod should use the image nginx:alpine.
Solution

```
Solution manifest file to create a Persistent Volume called log-volume as follows:-
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  hostPath:
    path: /opt/volume/nginx
```

then create a Persistent Volume Claim called log-claim as follows:----

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
  storageClassName: manual
```

Check the bind status of PV and PVC by running the following command:-
root@controlplane:~$ kubectl get pv,pvc
Now, create a new pod called logger with nginx:alpine image as follows:----

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: logger
# pod name
  name: logger
spec:
  containers:
  - image: nginx:alpine
    name: logger
    volumeMounts:
    - name: log
      mountPath: /var/www/nginx
  volumes:
  - name: log
    persistentVolumeClaim:
        claimName: log-claim
```

2. We have deployed a new pod called secure-pod and a service called secure-service. Incoming or Outgoing connections to this pod are not working. Troubleshoot why this is happening.
Make sure that incoming connection from the pod webapp-color are successful.
Important: Don't delete any current objects deployed.

Solution

Incoming or outgoing connections are not working because of network policy. In the default namespace, we deployed a default-deny network policy which is interrupting the incoming or outgoing connections.  Now, create a network policy called test-network-policy to allow the connections, as follows:-

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80
```

then check the connectivity from the webapp-color pod to the secure-pod:-
root@controlplane:~$ kubectl exec -it webapp-color -- sh
/opt # nc -v -z -w 5 secure-service 80



3. Create a pod called time-check in the dvl1987 namespace. This pod should run a container called time-check that uses the busybox image.
* Create a config map called time-config with the data TIME_FREQ=10 in the same namespace.
* The time-check container should run the command: while true; do date; sleep $TIME_FREQ;done and write the result to the location /opt/time/time-check.log.
* The path /opt/time on the pod should mount a volume that lasts the lifetime of this pod.

Create a namespace called dvl1987 by using the below command:-
$ kubectl create namespace dvl1987
Solution manifest file to create a configMap called time-config in the given namespace as follows:-

```
apiVersion: v1
data:
  TIME_FREQ: "10"
kind: ConfigMap
metadata:
  name: time-config
  namespace: dvl1987
```

Now, create a pod called time-check in the same namespace as follows:----

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: time-check
  name: time-check
  namespace: dvl1987
spec:
  volumes:
  - name: log-volume
    emptyDir: {}
  containers:
  - image: busybox
    name: time-check
    env:
    - name: TIME_FREQ
      valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
    volumeMounts:
    - mountPath: /opt/time
      name: log-volume
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log"
```



4. Create a new deployment called nginx-deploy, with one single container called nginx, image nginx:1.16 and 4 replicas. The deployment should use RollingUpdate strategy with maxSurge=1, and maxUnavailable=2.  Next upgrade the deployment to version 1.17.  Finally, once all pods are updated, undo the update and go back to the previous version.


Run the following command to create a manifest for deployment nginx-deploy and save it into a file:-
kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=4 --dry-run=client -oyaml > nginx-deploy.yaml
and add the strategy field under the spec section as follows:-
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
So final manifest file for deployment called nginx-deploy should looks like below:-

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        imagePullPolicy: IfNotPresent
        name: nginx
```

then run the kubectl apply -f nginx-deploy.yaml to create a deployment resource.  Now, upgrade the deployment's image version using the kubectl set image command:-
kubectl set image deployment nginx-deploy nginx=nginx:1.17
Run the kubectl rollout command to undo the update and go back to the previous version:-
kubectl rollout undo deployment nginx-deploy



5. Create a redis deployment with the following parameters:  Name of the deployment should be redis using the redis:alpine image. It should have exactly 1 replica.  The container should request for .2 CPU. It should use the label app=redis.  It should mount exactly 2 volumes.
a. An Empty directory volume called data at path /redis-master-data.  b. A configmap volume called redis-config at path /redis-master.  c. The container should expose the port 6379.   The configmap has already been created.


Solution manifest file to create a deployment redis as follows:-

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: redis-config
      containers:
      - image: redis:alpine
        name: redis
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: redis-config
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "0.2"
```



#### Practice test Docker Images
Base Operating System used by the python:3.6 image
```
docker run python:3.6 cat /etc/*release*
```

```
docker run -p 8383:8080 webapp-color:lite
```

```
k exec kube-apiserver-kubemaster1 -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```

#### Job

```
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  completions: 3
  backoffLimit: 25 # This is so the job does not quit before it succeeds.
  template:
    spec:
      containers:
      - name: throw-dice
        image: kodekloud/throw-dice
      restartPolicy: Never
```

#### Multi-Container PODs
```
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
  - name: lemon
    image: busybox
    command:
      - sleep
      - "1000"

  - name: gold
    image: redis
```

```
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```

https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
https://bit.ly/2EXYdHf

```
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  containers:
  - name: app
    image: kodekloud/event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume

  - name: sidecar
    image: kodekloud/filebeat-configured
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: DirectoryOrCreate
```

#### Init Containers

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ["sleep", "20"]
```

#### Job & CronJob

##### Job
```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  parallelism: 3
  completions: 3
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 25
```

##### CronJob
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │ 
# │ │ │ │ │
# * * * * *
```

#### Labs - Helm Concepts

Download the bitnami apache package under the /root directory.
```
helm pull --untar  bitnami/apache
```


#### Lab - API Versions/Deprecations

```
k proxy 8001&
curl localhost:8001/apis/authorization.k8s.io
```

#### Readiness & Liveness Probes

```
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 80
      periodSeconds: 1
```

```
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
```


#### Admission Controllers

#### Validating Admission Controllers

#### Mutating Admission Controllers


#### API Version
```
k explain deployment
```

#### Kubectl Convert

##### Install
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert"

chmod +x kubectl-convert

mv kubectl-convert /usr/local/bin/

kubectl-convert -h
```

```
kubectl convert -f <old-file> --output-version <new-api>
```



## Domains & Competencies
```
Application Design and Build (20%)
Define, build and modify container images
Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc.)
Understand multi-container Pod design patterns (e.g. sidecar, init and others)
Utilize persistent and ephemeral volumes

Application Deployment (20%)
Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)
Understand Deployments and how to perform rolling updates
Use the Helm package manager to deploy existing packages
Kustomize

Application Observability and Maintenance (15%)
Understand API deprecations
Implement probes and health checks
Use built-in CLI tools to monitor Kubernetes applications
Utilize container logs
Debugging in Kubernetes

Application Environment, Configuration and Security (25%)
Discover and use resources that extend Kubernetes (CRD, Operators)
Understand authentication, authorization and admission control
Understand Requests, limits, quotas
Understand ConfigMaps
Define resource requirements
Create & consume Secrets
Understand ServiceAccounts
Understand Application Security (SecurityContexts, Capabilities, etc.)

Services and Networking (20%)
Demonstrate basic understanding of NetworkPolicies
Provide and troubleshoot access to applications via services
Use Ingress rules to expose applications
```