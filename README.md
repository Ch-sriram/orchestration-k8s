# Orchestration &mdash; K8S

- This repository contains concepts and tools related to Kubernetes (K8S).
- The exercises are sourced from [LinkedIn Learning's Course &mdash; Learning Kubernetes by *Kim Schlesinger*](https://github.com/LinkedInLearning/learning-kubernetes-3212391/tree/main)

## Challenges

> The challenges are to be done after the content above it understood and practiced.

1. [Challenge 1: Create Your Own Deployment](#challenge-1-create-your-own-deployment)

## Table of Contents

- [Orchestration — K8S](#orchestration--k8s)
  - [Challenges](#challenges)
  - [Table of Contents](#table-of-contents)
  - [Get Started](#get-started)
  - [Spin-Up Clusters \& Interact With Them](#spin-up-clusters--interact-with-them)
  - [Create a Namepsace](#create-a-namepsace)
    - [Example Namespace](#example-namespace)
  - [Deploying An Application](#deploying-an-application)
    - [Creating Pods Using an Existing YAML Spec](#creating-pods-using-an-existing-yaml-spec)
    - [Check Health Using Event Logs](#check-health-using-event-logs)
    - [Application/Pod Verification Using `BusyBox`](#applicationpod-verification-using-busybox)
    - [Application/Pod Verification Using Application Logs](#applicationpod-verification-using-application-logs)
  - [Challenge 1: Create Your Own Deployment](#challenge-1-create-your-own-deployment)

## Get Started

- Install [`minikube`](https://minikube.sigs.k8s.io/docs/start/) &mdash; the page should load and show the correct command to download, based on which OS you've opened the page on.

## Spin-Up Clusters & Interact With Them

> ***Notes***:
>
> 1. `minikube` is used to create a cluster. Different cloud providers will have different commands for spinning-up/creating a cluster. `minikube` is a FOSS version of the cloud providers that isn't suitable for large-scale production enviornments, but works well for learning, and setting up applications at the smaller level for learning and experimentation.
> 2. `kubectl` is used to interact with a cluster once the cluster has been spun-up. This command is universal, and works even in a cloud provider host to interact with the cluster.

- Start the cluster using:

  ```sh
  # This would be different for different cloud providers
  minikube start
  ```

  You should see something as follows:

  ```terminal
  😄  minikube v1.38.1 on Ubuntu 22.04
  ✨  Automatically selected the docker driver. Other choices: ssh, none
  ❗  Starting v1.39.0, minikube will default to "containerd" container runtime. See #21973 for more info.
  📌  Using Docker driver with root privileges
  👍  Starting "minikube" primary control-plane node in "minikube" cluster
  🚜  Pulling base image v0.0.50 ...
  💾  Downloading Kubernetes v1.35.1 preload ...
      > preloaded-images-k8s-v18-v1...:  272.45 MiB / 272.45 MiB  100.00% 12.63 M^[[D
      > gcr.io/k8s-minikube/kicbase...:  519.58 MiB / 519.58 MiB  100.00% 15.81 M
  🔥  Creating docker container (CPUs=2, Memory=15900MB) ...
  🐳  Preparing Kubernetes v1.35.1 on Docker 29.2.1 ...
  🔗  Configuring bridge CNI (Container Networking Interface) ...
  🔎  Verifying Kubernetes components...
      ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
  🌟  Enabled addons: storage-provisioner, default-storageclass
  💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
  🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
  ```

- You can get cluster information using:

  ```sh
  kubectl cluster-info
  ```

  And you should see something like the following:

  ```terminal
  Kubernetes control plane is running at https://192.168.49.2:8443
  CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

  To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
  ```

- Once your cluster's ready, check the nodes running on the cluser using the following command:

  ```sh
  # Command should work everywhere unless `kubectl` isn't installed
  kubectl get nodes

  # if `kubectl` is not installed, use the same command in `minikube`'s context
  minikube kubectl -- get nodes
  ```

  You should see something like the following:

  ```terminal
  NAME       STATUS   ROLES           AGE   VERSION
  minikube   Ready    control-plane   21m   v1.35.1
  ```

- To look at the namespaces that get created by default:

  ```sh
  kubectl get namespaces
  ```

  By default, you should be seeing the following namespaces in your system [not necessarily the same ones, but some output should exist!]:

  ```terminal
  NAME              STATUS   AGE
  default           Active   22m
  kube-node-lease   Active   22m
  kube-public       Active   22m
  kube-system       Active   22m
  ```

  > ***NOTE***: `namespaces` are way to isolate and manage applications & services that you want to remain separate.

- To look at the pods that are installed, when you spin-up a minikube cluster:

  ```sh
  kubectl get pods -A # -A: list pods in all namespaces
  ```

  Output should be something like this:

  ```terminal
  NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
  kube-system   coredns-7d764666f9-58mp6           1/1     Running   0             26m
  kube-system   etcd-minikube                      1/1     Running   0             26m
  kube-system   kube-apiserver-minikube            1/1     Running   0             26m
  kube-system   kube-controller-manager-minikube   1/1     Running   0             26m
  kube-system   kube-proxy-ssgpq                   1/1     Running   0             26m
  kube-system   kube-scheduler-minikube            1/1     Running   0             26m
  kube-system   storage-provisioner                1/1     Running   1 (25m ago)   26m
  ```

- To look at the services that are installed, when you spin-up a minikube cluster:

  ```sh
  kubectl get services -A # -A: list services in all namespaces
  ```

  Output should look something like:

  ```terminal
  NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
  default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  29m
  kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   29m
  ```

  > ***NOTE***: `services` act as node balances within a cluster and they direct traffic to `pods`.

[Go 🆙](#table-of-contents)

## Create a Namepsace

- Kubernetes `namespace`s lets you isolate organize your workloads.
  - Ex: If you've different environments for your application and services, you can organize the different environments of your application using namespaces like `"dev"` [for development] and `"prod"` [for production].

### Example Namespace

- The following yaml is available at [`nameppace.yml`](./Ex_Files_Learning_Kubernetes/Exercise_Files/03_02_Begin/namespace.yaml)

  ```yml
  ---
  apiVersion: v1
  kind: Namespace
  metadata:
    name: development # this is what's important; this namespace is called `development`
  ```

- To create a namespace from the aforementioned file, you run `kubectl` as:

  ```sh
  kubectl apply -f /path/to/namespace.yaml
  ```

  You should see something like this:

  > ```terminal
  > namespace/development created
  > ```

- Then check the created namespace using:

  ```sh
  kubectl get namespaces
  ```

  > You should something as follows:
  >
  > ```terminal
  > NAME              STATUS   AGE
  > default           Active   26h
  > development       Active   2s        <----  NEWLY CREATED
  > kube-node-lease   Active   26h
  > kube-public       Active   26h
  > kube-system       Active   26h
  > ```

- Because kubernetes manifests are written in YAML, it's possible to define multiple resources in one file by separating them with 3 horizontal dashes `-` as follows:

  ```yml
  ---
  apiVersion: v1
  kind: Namespace
  metadata:
    name: development # this is what's important; this namespace is called `development`
  ---         # <--- signifies a new resource definition in k8s
  apiVersion: v1
  kind: Namespace
  metadata:
    name: production # this is what's important; this namespace is called `production`
  ```

  > As can be seen here, there are 2 namepsaces defined here in the same manifest because YAML allows this by using the 3 horizontal dashes `---`.

- Run the apply command again:

  ```sh
  kubectl apply -f /path/to/namespace.yaml
  ```

  You should see something like this:

  > ```terminal
  > namespace/development unchanged
  > namespace/production created
  > ```

  Get the namespaces:

  ```sh
  kubectl get namespaces
  ```

  You should see the following:

  > ```terminal
  > NAME              STATUS   AGE
  > default           Active   27h
  > development       Active   45m
  > kube-node-lease   Active   27h
  > kube-public       Active   27h
  > kube-system       Active   27h
  > production        Active   118s
  > ```

- To ***delete*** the namespaces that were just created, use the following command:

  ```sh
  kubectl delete -f /path/to/namespace.yaml
  ```

  You should see the following:

  > ```terminal
  > namespace "development" deleted
  > namespace "production" deleted
  > ```

[Go 🆙](#table-of-contents)

## Deploying An Application

- K8S is designed to make your applications highly available, meaning that there are multiple replicas of your application running at the same time, so that if one stops working, there are at least two others accepting traffic.
- `Pods` are the k8s resource that run our applications and microservices, and one way to ensure that an application is highly available, is to organize your pods using a kubernetes deployment.

### Creating Pods Using an Existing YAML Spec

- You can find the following YAML spec in [`deployment.yaml`](./Ex_Files_Learning_Kubernetes/Exercise_Files/03_03/deployment.yaml)

  ```yml
  ---
  apiVersion: apps/v1         # API group to which the requests are being sent to
  kind: Deployment            # Kind of k8s object we want to create, which is "Deployment" object.
  metadata:
    name: pod-info-deployment # Name given to the deployment
    namespace: development    # Namespace where the pods will be created at
    labels:
      app: pod-info           # Labelling all the pods in this group, with the app name as `pod-info`
  spec:
    replicas: 3               # How many replicas of the container we want to run is specified here
    selector:
      matchLabels:
        app: pod-info
    template:
      metadata:
        labels:
          app: pod-info
      spec:
        containers:
          - name: pod-info-container
            image: kimschles/pod-info-app:latest    # Pulls the latest version of `pod-info-app`, and then directs traffic to port 3000
            ports:
              - ctonainerPort: 3000
            env:
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.namespace
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: status.podIP
  ```

- Before creating the deployment, let's make sure that we've the `development` namespace:

  ```sh
  kubectl get ns # same as: kubectl get namespaces
  ```

- Create the `development` namespace using [`nameppace.yml`](./Ex_Files_Learning_Kubernetes/Exercise_Files/03_02_Begin/namespace.yaml):

  ```sh
  kubectl apply -f /path/to/namespace.yaml
  ```

  You should see something like this:

  > ```terminal
  > namespace/development created
  > ```

- To run the deployment, simply do as follows:

  ```sh
  kubectl apply -f /path/deployment.yaml
  ```

  You should see something like:

  > ```terminal
  > deployment.apps/pod-info-deployment created
  > ```

- To confirm, list all the containers deployed in the `development` namespace:

  ```sh
  kubectl get deployments -n development        # -n: namespace
  ```

  Output should be:

  > ```terminal
  > NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
  > pod-info-deployment   3/3     3            3           91s
  > ```

- To check the pods created by the deployment, run:

  ```sh
  kubectl get pods -n development               # -n: namespace
  ```

  Output should be something like:

  > ```terminal
  > NAME                                   READY   STATUS              RESTARTS   AGE
  > pod-info-deployment-64f9d5546f-6rqsw   0/1     ContainerCreating   0          5s
  > pod-info-deployment-64f9d5546f-7n8ks   0/1     ContainerCreating   0          5s
  > pod-info-deployment-64f9d5546f-dp6lf   1/1     Running             0          5s
  > ```
  >
  > There exactly 3 pods! (as `replicas: 3` in [`development.yaml`](./Ex_Files_Learning_Kubernetes/Exercise_Files/03_03/deployment.yaml#L10) manifest)

- When it's configured to be `3` replicas, there will always be those 3 replicas of the same application, and that can be shown using:

  ```sh
  # syntax: kubcetl delete pod <pod-name> -n <namespace>
  kubectl delete pod pod-info-deployment-64f9d5546f-6rqsw -n development
  ```

  O/P:

  ```terminal
  pod "pod-info-deployment-64f9d5546f-6rqsw" deleted from development namespace
  ```

  Now, if we list the pods in `development` namespace, there would be 3 pods for sure:

  ```sh
  kubectl get pods -n development
  ```

  O/P:

  ```terminal
  NAME                                   READY   STATUS    RESTARTS   AGE
  pod-info-deployment-64f9d5546f-7n8ks   1/1     Running   0          9m49s
  pod-info-deployment-64f9d5546f-dp6lf   1/1     Running   0          9m49s
  pod-info-deployment-64f9d5546f-pqm6m   1/1     Running   0          2m9s
  ```
  
  > As you can see! there are no pods that were destroyed, but a new one got spun-up in no time.

[Go 🆙](#table-of-contents)

### Check Health Using Event Logs

- K8S saves the event log when a new pod is created during a deployment, so that anyone can troubleshoot the pod in case of a failure.
- Get the pod whose event log you'd like to view:

  ```sh
  kubectl get pods -n development
  ```

  Output can be like this:

  > ```terminal
  > NAME                                   READY   STATUS    RESTARTS   AGE
  > pod-info-deployment-64f9d5546f-7n8ks   1/1     Running   0          19m
  > pod-info-deployment-64f9d5546f-dp6lf   1/1     Running   0          19m
  > pod-info-deployment-64f9d5546f-pqm6m   1/1     Running   0          11m
  > ```

  We can copy any pod's name we want, and check its event log.

- To check the event log of a pod, use:

  ```sh
  kubectl describe pod pod-info-deployment-64f9d5546f-7n8ks -n development
  ```

  You should see something like the following:

  ```terminal
  Name:             pod-info-deployment-64f9d5546f-7n8ks
  Namespace:        development
  Priority:         0
  Service Account:  default
  Node:             minikube/192.168.49.2
  Start Time:       Tue, 24 Mar 2026 20:14:11 +0530
  Labels:           app=pod-info
                    pod-template-hash=64f9d5546f
  Annotations:      <none>
  Status:           Running
  IP:               10.244.0.5
  IPs:
    IP:           10.244.0.5
  Controlled By:  ReplicaSet/pod-info-deployment-64f9d5546f
  Containers:
    pod-info-container:
      Container ID:   docker://1493115e2664fd5d62fc19119d9a35935bf9512d54f64b68c772e7b0fa610efb
      Image:          kimschles/pod-info-app:latest
      Image ID:       docker-pullable://kimschles/pod-info-app@sha256:fa4f33bc2301bb242bdd078ac206d0e379dfed2e225d46a6952ff444ae6f4a7a
      Port:           3000/TCP
      Host Port:      0/TCP
      State:          Running
        Started:      Tue, 24 Mar 2026 20:14:16 +0530
      Ready:          True
      Restart Count:  0
      Environment:
        POD_NAME:       pod-info-deployment-64f9d5546f-7n8ks (v1:metadata.name)
        POD_NAMESPACE:  development (v1:metadata.namespace)
        POD_IP:          (v1:status.podIP)
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-94qf7 (ro)
  Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       True 
    ContainersReady             True 
    PodScheduled                True 
  Volumes:
    kube-api-access-94qf7:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      Optional:                false
      DownwardAPI:             true
  QoS Class:                   BestEffort
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  21m   default-scheduler  Successfully assigned development/pod-info-deployment-64f9d5546f-7n8ks to minikube
    Normal  Pulling    21m   kubelet            spec.containers{pod-info-container}: Pulling image "kimschles/pod-info-app:latest"
    Normal  Pulled     21m   kubelet            spec.containers{pod-info-container}: Successfully pulled image "kimschles/pod-info-app:latest" in 2.338s (4.873s including waiting). Image size: 213980203 bytes.
    Normal  Created    21m   kubelet            spec.containers{pod-info-container}: Container created
    Normal  Started    21m   kubelet            spec.containers{pod-info-container}: Container started
  ```

  > There are no issues with the pod, but, most issues of the pod during the first few minutes of the deployment.

[Go 🆙](#table-of-contents)

### Application/Pod Verification Using `BusyBox`

- Once you deploy, you might want to verify that the application is working as expected.
- One way to verify and check, is to use a tool called **BusyBox** [contains many utilities like `awk`, `wget`, `date`, `whoami`, etc]. It's a great tool for debugging, and troubleshooting applications on a linux environment, and k8s runs on linux.
- Since we just want to debug and verify our application's working, we can just use our default namespace, and create only 1 replica of the `BusyBox` pod, from the file in [`busybox.yaml`](./Ex_Files_Learning_Kubernetes/Exercise_Files/03_05/busybox.yaml).

  ```yml
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: busybox
    namespace: default
    labels:
      app: busybox
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
          - name: busybox-container
            image: busybox:latest
            command: ['/bin/sh', '-c', '--'] # This keeps the container running, otherwise, the container will start and terminate
            args: ['while true; do sleep 30; done']
            resources:
              requests:
                cpu: 30m
                memory: 60Mi
              limits:
                cpu: 100m
                memory: 128Mi
  ```

  ```sh
  kubectl apply -f /path/to/busybox.yaml
  ```

  Check whether the container was created or not:

  ```sh
  kubectl get pods
  ```

  O/P should be:

  ```terminal
  NAME                       READY   STATUS              RESTARTS   AGE
  busybox-65c9d65546-wzvqj   0/1     ContainerCreating   0          6s
  ```

  > Remember, whenever you use the command `kubectl get pods`, it always gets the pods running in `default` namespace.
  > To explicitly get the pods running in a namespace, you've to use the command - `kubectl get pods -n <namespace-name>`

- Now get the pods running in `development` namespace using the `wide` option to show extra information like IP addresses, restarts, node, etc:

  ```sh
  kubectl get pods -n development -o wide
  ```

  O/P:

  ```terminal
  NAME                                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
  pod-info-deployment-64f9d5546f-7n8ks   1/1     Running   0          24h   10.244.0.5   minikube   <none>           <none>
  pod-info-deployment-64f9d5546f-dp6lf   1/1     Running   0          24h   10.244.0.4   minikube   <none>           <none>
  pod-info-deployment-64f9d5546f-pqm6m   1/1     Running   0          24h   10.244.0.7   minikube   <none>           <none>
  ```

- Login to the busybox pod using the following `exec` command [similar to how we login to a container using `docker`]:

  ```sh
  kubectl exec -it busybox-65c9d65546-wzvqj -- /bin/sh
  ```

  You should be able to see a new shell prompt as follows:

  ```terminal
  / #
  ```

  > You're successfully logged into busybox pod.

- In the busybox pod, try and `wget` on one of the IPs for the pods in `development` namespace as follows:

  ```terminal
  / # wget 10.244.0.5
  ```

  You should see something like the following:

  ```terminal
  / # wget 10.244.0.5
  Connecting to 10.244.0.5 (10.244.0.5:80)
  wget: can't connect to remote host (10.244.0.5): Connection refused
  / #
  ```

  That's because the app is originally deployed on port `3000`, as mentioned in [`deployment.yamlL23`](./Ex_Files_Learning_Kubernetes/Exercise_Files/03_03/deployment.yamlL23).

- Now if you try to use `wget` for the same IP, on port `3000`, as follows:

  ```terminal
  / # wget 10.244.0.5:3000
  ```

  O/P should be the following:

  ```terminal
  / # wget 10.244.0.5:3000
  Connecting to 10.244.0.5:3000 (10.244.0.5:3000)
  saving to 'index.html'
  index.html           100% |*********************************************************************************************************************************|   103  0:00:00 ETA
  'index.html' saved
  ```

  > `wget` saved a file called `index.html` in the same directory.
  > To look into the file, you can run `cat index.html`. It just returns the pod name, namespace, and IP like: `{"pod_name":"pod-info-deployment-64f9d5546f-7n8ks","pod_namespace":"development","pod_ip":"10.244.0.5"}`.

[Go 🆙](#table-of-contents)

### Application/Pod Verification Using Application Logs

- We can also check whether the application is working or not, using application logs.
- To do so, we need to use the following command [but first, get the name of the pod whose logs you'd like to inspect using `kubectl get pods -n <namespace-name>`]:

  ```sh
  kubectl logs pod-info-deployment-64f9d5546f-7n8ks -n development
  ```

  O/P should be:

  ```terminal
  undefined
  Example app listening on port 3000
  ```

[Go 🆙](#table-of-contents)

## Challenge 1: Create Your Own Deployment

1. Create a new deployment in a file called `quote.yaml`
2. Name the deployment and name the app label `quote-service`
3. Use the `development` namespace
4. Name the container `quote-container`
5. Run `2` replicas
6. Use the image `datawire/quote:0.5.0`
7. Set the container to accept traffic at port `8080`
8. Create the pods using `kubectl apply -f quote.yml`
9. [OPTIONAL] Use `BusyBox` to test that the application can accept traffic from inside the cluster

> Solution [along with an explanation] can be found at [`challenge-solutions/challenge-1__create-your-own-deployment/`](.challenge-solutions/challenge-1__create-your-own-deployment/).

[Go 🆙](#table-of-contents)
