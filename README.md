# Orchestration &mdash; K8S

- This repository contains concepts and tools related to Kubernetes (K8S).
- The exercises are sourced from [LinkedIn Learning's Course &mdash; Learning Kubernetes by *Kim Schlesinger*](https://github.com/LinkedInLearning/learning-kubernetes-3212391/tree/main)

## Table of Contents

- [Get Started](#get-started)
- [Spin-Up Clusters \& Interact With Them](#spin-up-clusters--interact-with-them)
- [Create a Namepsace](#create-a-namepsace)
  - [Example Namespace](#example-namespace)

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
