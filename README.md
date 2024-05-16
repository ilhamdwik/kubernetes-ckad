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



#### Test Labs Udemy 2

1. Task
Welcome to the KodeKloud CKAD Lightning Lab - Part 2!

NOTE: You can toggle between the questions but make sure that that you click on END EXAM before the the timer runs out.
While this test environment is valid for 60 minutes, challenge yourself and try to complete all 5 questions within 30 minutes! To pass, correctly complete at least 4 out of 5 questions. Good Luck!!!


We have deployed a few pods in this cluster in various namespaces. Inspect them and identify the pod which is not in a Ready state. Troubleshoot and fix the issue.

Next, add a check to restart the container on the same pod if the command ls /var/www/html/file_check fails. This check should start after a delay of 10 seconds and run every 60 seconds.

You may delete and recreate the object. Ignore the warnings from the probe.

Solution
The pod nginx1401 is not in a Ready state as the Readiness Probe has failed. Here is the solution YAML file:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx1401
  namespace: dev1401
spec:
  containers:
  - image: kodekloud/nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 9080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /
        port: 9080    
    livenessProbe:
      exec:
        command:
        - ls
        - /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60
```


2. Task
Create a cronjob called dice that runs every one minute. Use the Pod template located at /root/throw-a-dice. The image throw-dice randomly returns a value between 1 and 6. The result of 6 is considered success and all others are failure.

The job should be non-parallel and complete the task once. Use a backoffLimit of 25.

If the task is not completed within 20 seconds the job should fail and pods should be terminated.

You don't have to wait for the job completion. As long as the cronjob has been created as per the requirements.

Solution
Use the following YAML file to create the cronjob:

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dice
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 1
      backoffLimit: 25 # This is so the job does not quit before it succeeds.
      activeDeadlineSeconds: 20
      template:
        spec:
          containers:
          - name: dice
            image: kodekloud/throw-dice
          restartPolicy: Never
```


3. Task
Create a pod called my-busybox in the dev2406 namespace using the busybox image. The container should be called secret and should sleep for 3600 seconds.

The container should mount a read-only secret volume called secret-volume at the path /etc/secret-volume. The secret being mounted has already been created for you and is called dotfile-secret.

Make sure that the pod is scheduled on controlplane and no other node in the cluster.


Solution
Use the following YAML file to create the pod:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-busybox
  name: my-busybox
  namespace: dev2406
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  nodeSelector:
    kubernetes.io/hostname: controlplane
  containers:
  - command:
    - sleep
    args:
    - "3600"
    image: busybox
    name: secret
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

4. Task
Create a single ingress resource called ingress-vh-routing. The resource should route HTTP traffic to multiple hostnames as specified below:

The service video-service should be accessible on http://watch.ecom-store.com:30093/video

The service apparels-service should be accessible on http://apparels.ecom-store.com:30093/wear

Here 30093 is the port used by the Ingress Controller

Solution
Use the following YAML to create the ingress resource:

```
---
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: ingress-vh-routing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: watch.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/video"
        backend:
          service:
            name: video-service
            port:
              number: 8080
  - host: apparels.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/wear"
        backend:
          service:
            name: apparels-service
            port:
              number: 8080
```


5. Task
A pod called dev-pod-dind-878516 has been deployed in the default namespace. Inspect the logs for the container called log-x and redirect the warnings to /opt/dind-878516_logs.txt on the controlplane node

Solution
Run the command:

```
kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt
```



Task
Create a new Ingress Resource for the service my-video-service to be made available at the URL: http://ckad-mock-exam-solution.com:30093/video.

To create an ingress resource, the following details are: -

annotation: nginx.ingress.kubernetes.io/rewrite-target: /

host: ckad-mock-exam-solution.com

path: /video

Once set up, the curl test of the URL from the nodes should be successful: HTTP 200

Solution
Create an ingress resource manifest file using the imperative command:-

kubectl create ingress ingress --rule="ckad-mock-exam-solution.com/video*=my-video-service:8080" --dry-run=client -oyaml > ingress.yaml
And then add the rewrite-target annotation.

The final manifest file will look like this.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: ingress
spec:
  rules:
  - host: ckad-mock-exam-solution.com
    http:
      paths:
      - backend:
          service:
            name: my-video-service
            port:
              number: 8080
        path: /video
        pathType: Prefix
```


Task
Create a job called whalesay with image docker/whalesay and command "cowsay I am going to ace CKAD!".

completions: 10

backoffLimit: 6

restartPolicy: Never

This simple job runs the popular cowsay game that was modifed by docker…

Solution
Solution manifest file to create a job called whalesay as follows:-

```
apiVersion: batch/v1
kind: Job
metadata:
  name: whalesay
spec:
  completions: 10
  backoffLimit: 6
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - sh 
        - -c
        - "cowsay I am going to ace CKAD!"
        image: docker/whalesay
        name: whalesay
      restartPolicy: Never
```


Task
Create a pod called multi-pod with two containers.

Container 1:
name: jupiter, image: nginx

Container 2:
name: europa, image: busybox
command: sleep 4800

Environment Variables:

Container 1:

type: planet

Container 2:

type: moon

Solution
Solution manifest file to create a multi pod containers called multi-pod as follows:-

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - image: nginx
    name: jupiter
    env:
    - name: type
      value: planet
  - image: busybox
    name: europa
    command: ["/bin/sh","-c","sleep 4800"]
    env:
     - name: type
       value: moon
```


Task
Create a PersistentVolume called custom-volume with size: 50MiB reclaim policy:retain, Access Modes: ReadWriteMany and hostPath: /opt/data

Solution
Solution manifest file to create a Persistent Volume called custom-volume as follows:-

```
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: custom-volume
spec:
  accessModes: ["ReadWriteMany"]
  capacity:
    storage: 50Mi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /opt/data
```



#### Imperative Commands

Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80.


Try to do this with as few steps as possible.

```
kubectl run httpd --image=httpd:alpine --port=80 --expose
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

#### Job
```
k create job throw-dice-job --image=kodekloud/throw-dice
```
```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: throw-dice-job
spec:
  backoffLimit: 50
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice-job
        resources: {}
      restartPolicy: Never
status: {}
```

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

#### Security Context

pod ubuntu-sleeper to run as Root user and with the SYS_TIME capability.
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
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

##### Labs - Admission Controllers

Check enable-admission-plugins in kube-apiserver help options
```
kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'
```

Which admission controller is enabled in this cluster which is normally disabled?
```
grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml
```

Disable DefaultStorageClass admission controller
```
nano /etc/kubernetes/manifests/kube-apiserver.yaml

and add line

- --disable-admission-plugins=DefaultStorageClass
```

Since the kube-apiserver is running as pod you can check the process to see enabled and disabled plugins.
```
ps -ef | grep kube-apiserver | grep admission-plugins
```

#### Validating Admission Controllers

#### Mutating Admission Controllers

#### ConfigMaps

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: kodekloud/simple-webapp-color
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: webapp-config-map
              key: APP_COLOR
  restartPolicy: Never
```

#### Secrets

pod use secret
```
apiVersion: v1 
kind: Pod 
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default 
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: db-secret
```

#### ServiceAccount

```
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: web-dashboard
    namespace: default
  spec:
    selector:
      matchLabels:
        name: web-dashboard
    template:
      metadata:
        creationTimestamp: null
        labels:
          name: web-dashboard
      spec:
        containers:
        - env:
          - name: PYTHONUNBUFFERED
            value: "1"
          image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
          imagePullPolicy: Always
          name: web-dashboard
          ports:
          - containerPort: 8080
            protocol: TCP
        serviceAccount: dashboard-sa
```

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

#### Netpol

Create a network policy to allow traffic from the Internal application only to the payroll-service and db-service.


Use the spec given below. You might want to enable ingress traffic to the pod to test your rules in the UI.

Also, ensure that you allow egress traffic to DNS ports TCP and UDP (port 53) to enable DNS resolution from the internal pod.

Policy Name: internal-policy

Policy Type: Egress

Egress Allow: payroll

Payroll Port: 8080

Egress Allow: mysql

MySQL Port: 3306


Solution manifest file for a network policy internal-policy as follows:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```

Note: We have also allowed Egress traffic to TCP and UDP port. This has been added to ensure that the internal DNS resolution works from the internal pod.

Remember: The kube-dns service is exposed on port 53:


#### Ingress

If the requirement does not match any of the configured paths in the Ingress, to which service are the requests forwarded?
```
kubectl get deploy ingress-nginx-controller -n ingress-nginx -o yaml
```

default backend service
```
- --default-backend-service=app-space/default-backend-service
```

ingress pay-service
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
           name: pay-service
           port:
            number: 8282
```


#### Service Account - Set Service Account to deployment
```
kubectl set serviceaccount deployment nginx-deployment serviceaccount1
```

#### Taint and Tolerations
```
kubectl taint nodes foo dedicated=special-user:NoSchedule
```

##### Remove taint node on controlplane
```
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```


#### Label Node
```
kubectl label node node01 color=blue
```

#### Node Affinity
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
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