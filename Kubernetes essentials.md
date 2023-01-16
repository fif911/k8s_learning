# Kubernetes essentials

[Kubernetes concepts](https://kubernetes.io/docs/concepts/)

## Components

### Cluster

*A Kubernetes cluster is a set of nodes that run containerized applications.*

(Containerizing applications packages an app
with its dependences and some necessary services.) Kubernetes clusters allow containers to run across multiple machines
and environments: virtual, physical, cloud-based, and on-premises.

* Kubernetes clusters are comprised of one master node and a number of worker nodes.
* For production and staging, the cluster is distributed across multiple worker nodes. For testing, the components can
  all run on the same physical or virtual node.

![](https://d33wubrfki0l68.cloudfront.net/283cc20bb49089cb2ca54d51b4ac27720c1a7902/34424/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

### Control plane

*The container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of
containers.*

* In production environments, the control plane usually runs across multiple computers. Its components can be scaled
  horizontally (except for *ectd*).
* It is usually formed by it is formed by 1-3+ Master nodes

This layer is composed by many different components, such as (but not restricted to):

* [etcd](https://kubernetes.io/docs/concepts/overview/components/#etcd)
* [*kube-apiserver* API Server](https://kubernetes.io/docs/concepts/overview/components/#kube-scheduler)
* [*kube-schedule* Scheduler](https://kubernetes.io/docs/concepts/overview/components/#kube-scheduler)
* *kube-controller-manager* Controller Manager - controllers are control loops that watch the state of your cluster,
  then make or request changes where needed. Each controller tries to move the current cluster state closer to the
  desired state. Controllers watch the shared state of your cluster through the apiserver (part of the Control Plane).
* *cloud-controller-manager* Cloud Controller Manager

####          

![](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

* Note how Control plane is designed to scale horizontally

### Cluster's Master node

*Physical or Virtual Machine
that doesn’t run user workloads directly
but manages the cluster*

#### What does it do

The master node controls the state of the cluster; for example, which applications are running and their corresponding
container images. The master node is the origin for all task assignments. It coordinates processes such as:

* Scheduling and scaling applications
* Maintaining a cluster’s state
* Implementing updates

### Worker node

*Nodes are the machines, either VMs or physical servers, where Kubernetes place Pods to execute*

Every Kubernetes Node runs at least:

* **Kubelet**, a process responsible for communication between the Kubernetes control plane and the Node; it manages the
  Pods and the containers running on a machine.
* **A container runtime** (like Docker) responsible for pulling the container image from a registry, unpacking the
  container, and running the application.

![](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

### Pod

*The Pods are groups of containers that share networking and storage resources from the same node*

Each Pod is assigned an IP address, and all the containers in the Pod share the same storage, IP address, and port
space (network namespace).

## [Networking](https://kubernetes.io/docs/concepts/services-networking/)

### Service

An abstract way to expose an application running on a set of Pods as a network service. Kubernetes gives Pods their own
IP addresses and a single DNS name for a set of Pods, and can load-balance across them.

A Service is an abstraction which defines a logical set of Pods and a policy by which to access them (sometimes this
pattern is called a micro-service). The set of Pods targeted by a Service is usually determined by a selector.

#### [Defining a Service](https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service)

### Ingress(доступ)

#### [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

In order for the Ingress resource to work, the cluster must have an ingress controller running.

Unlike other types of controllers which run as part of the kube-controller-manager binary, Ingress controllers are not
started automatically with a cluster. Use this page to choose the ingress controller implementation that best fits your
cluster.

Kubernetes as a project supports and maintains AWS, GCE, and nginx ingress controllers