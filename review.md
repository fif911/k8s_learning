# Exam review

____
## Sample exam questions
1. What is the primary purpose of **cgroups**?  
- To limit resources used by processes, it is a Linux feature not a docker or kubernetes feature.
2. How many instances of FROM can you have in a dockerfile?  
- One or more
3. What is the order of precedence for values being overridden in Kubernetes Helm charts?
- The values in the values.yml file of the chart
- The values in the values.yml of the parent chart if there is one
- The values from a file supplied with the -f flag when installing or upgrading a helm application
- The values that were passed with the --set flag when interacting with a helm application
4. What can you do with a Horizontal Pod Autoscaler?
- Set the threshold CPU usage that if it is exceeded will cause the autoscaler to increase the number of pods
- Set the threshold Memory usage that if it is exceeded will cause the autoscaler to increase the number of pods
- Set the minimum number of pods that the autoscaler should create
- Set the maximum number of pods that the autoscaler should create
5. Which ways can Pods get data from ConfigMaps?
- Environment variables that are set in the Pod yaml file of the using the configMapRef or configMapKeyRef or etc.
- Read-only configuration files in a volume that the pod can mount and read
- Using container code that calls the Kubernetes REST API to get the data from the ConfigMap object
6. What causes a deployment rollout?
- Changing anything in the template area of the spec of a deployment
- Any  changes outside this scope will not trigger a deployment rollout
7. Kubernetes jobs run some code until completion, if you specify backOffLimit at 4, completions at 3, and parallelism at 2, when will the job complete?
- When 3 of the created containers have completed the job successfully or when 3 containers have retried to do and failed to do the job 4 times. Since parallelism is 2 we expect that 2 pods will be created at a time
    - [backOffLimit explained](https://stackoverflow.com/posts/60220748/timeline)
    - Use `spec.backoffLimit` to specify the number of retries before considering a Job as failed. The back-off limit is set to 6 by default.
8. What are some true statements about a service of type LoadBalancer?
- A LoadBalancer will allocate a ClusterIP
- A LoadBalancer will allocate a NodePort
- A LoadBalancer in an object that references some external LoadBalancer application
9. What happens when you make a Service of type NodePort?
- A port is opened on every Node including Masters and Workers
- That allows you to access that service by hitting any node at the specified port
10. What are the main differences between Volumes and Persistent Volumes?
- Volumes are bound to the lifecycle of the Pods, Persistent Volumes are created independently and the data is kept on the host machine after the Pod is deleted or dead or anything else.
- Volumes are mounted in all containers of the Pod, Persistent Volumes are not mounted directly to containers in a Pod (you instead have to create a Persistent Volume claim that gives access to the containers of a Pod to the data)

____
# Lecture 1 review

## Containers vs Virtual Machines
### Virtual machines
    - Isolation of the resources of various vms and the host system is achieved by loading a Guest OS on top of the Hypervisor element of the Host OS
        - This has a lot of overhead since it a full OS that is using resources to run
    - Examples of Hypervisors
        - VMWare ESXi
        - Oracle VirtualBox
        - KVM
        - IBM zVM
        - Xen Hypervisor
        - QEMU
        - Microsoft Windows Hyper-V
        - libvirt (Virtualization library)
### Containers
    - Resources are separated on the host operating system itself, different OSs achieve this in different ways. On Linux cgroups and Linux namespaces are usually used.
        - This means that each container will only need the resources it requires for it's application and not an entire OS which has a lot more moving parts than the application itself
            - Microsoft supports containers this way via Hyper-V
## Linux kernel features that enable containers
- Control groups (cgroups) (this is a type of Linux namespace)
- Namespaces
    - A namespace in Linux is a mechanism that allows the separation of a common resource among processes, i.e. share a global resource with processes in way that doesn't expose the entire resource and how it is being accessed by other jobs or processes
    - There are currently 8 kinds of namespace
        - Mount namespaces (mnt) isolate the set of mount points that a process sees
        - Interprocess Communication Namespace (ipc) isolate message queues, semaphores, shared memory
        - Unix Time Sharing namespace (UTS) isolates the hostname and NIS domain name of a process
        - Process ID namespace (pid) isolate the process ID number space
        - Network namespace (net) isolation of network devices, IP adresses, IP routing tables, /proc/net directory, port numbers etc
        - User ID namespace (user) isolate the user and group ID number spaces. A process can have a normal user ID outside a user namespace while having a user ID of 0 (root) inside the namespace
        - Control group namespace (cgroup) virtualize the view of a process' cgroups as seen via /proc/[pid]/cgroup and /proc/[pid]/mountinfo
            - cgroups limit the usage of resources by processes
            - cgroups can be managed by interactive the the files under /sys/fs/cgroup
            - OOM killer works with cgroups to kill processes when memory gets exhausted
        - Time namespace virtualize the values of two system clocks
    - Using the util-linux package you can list Linux namespaces
    - If you know the PID of a process then you can list the namespaces it belongs to by listing the contents of /proc/PID/ns
    - Kubernetes makes use of cgroups under the following directory /sys/fs/cgroup/cpu/kubepods
- Union Filesystems (UnionFS)
    - It is a file system that creates a union of separate directories known as branches
    - OverlayFS is a an implementation of UnionFS, it uses a hierarchical directory tree
        - An upper and lower tree exist
        - When there is a file in both the upper tree it takes precedence and the lower file is hidden
        - With directories the upper and lower trees are merged with the above also taking place
    - Docker uses UnionFS variants, including AUFS and OverlayFS
    - Docker provides 2 storage drives overlay and overlay2
    - Overlay2 is more efficient in terms of inode utilization
    - Overlay2 supports up to 128 lower OverlayFS layers
## Container Runtimes
- Kubernetes has a Container Runtime Interface (CRI) which is an API that enables kubelet to use different container runtimes
- Container runtimes supported by Kubernetes:
    - containerd
    - CRI-O
    - docker
- The docker runtime will be removed from Kubernetes at some point. This is because even though it is based on containerd it introduces some features that are not compliant with the CRI, requiring Kubernetes to add a shim to run the docker runtime.
## Docker
### Docker ecosystem
    - Docker Engine
        - Client server application 
        - That has a server with a long running dockerd process
        - Has APIs that programs can use to talk to dockerd
        - CLI interface that can be used to access dockerd
        - Swarm mode for natively managing a cluster of Docker Engines called a swarm
    - Docker Compose
        - Tool that allows you to spring up multi container applications in an organised manner using yaml files
        - NOT an orchestration system such as Kubernetes
    - Docker Desktop
        - Application for Mac or Windows for making and sharing images, and managing containers 
    - Docker Hub
        - Container Image repositories hosting
        - Used to share container images
        - Repository of container images with contributions from community developers, open source projects and anyone who can make an account
        - Public repos are free, private repos are available with a subscription
### Docker images
    - Docker images are built out of layers
    - Images can be created from declarative files called dockerfiles
    - Most commands in a dockerfile result in a new layer being added to the image
    - Many commands
        - FROM, RUN, COPY, CMD, ....
        - FROM: Defines the base image if there is only one, then the last one is the base image. More than one can be used so that you can use an image for building the next image
        - RUN: Used to run commands within the image being created, adds a new layer to the image. Can have 0 or more
        - COPY: Copies files from the host system to the image filesystem, can have 0 or more. The destination path is either an absolute path or relative to WORKDIR if that has been set
        - EXPOSE: Informs Docker that the container listens on the specified network ports at runtime. By default nothing is done with the ports. They are not open from the host machine, so hostIp:port will not work. You have to map the container port to a host a port, done using the -p tag during a docker run. Can have 0 or more
        - CMD: Used to define the default execution command for a container. Can only have 1. If one is not specified or it is specified but doe not contain an executable then ENTRYPOINT must be defined. The parameters of the executable part of the CMD command can be overridden by passing more arguements during the docker run command
        - ENTRYPOINT: The same as CMD but no overriding possible. Overrides the things defined in a CMD command if one is present.
        - WORKDIR: Sets the working directory for all other dockerfile layer commands. Can have multiple WORKDIR commands. The path of the first WORKDIR must be absolute, all other WORKDIR commands will be relative to the previous one unless they are written as absolute paths.
    - Image is created by running the docker build command on a dockerfile
    - After docker builds the image all the layers are read only
    - When the image is run a writeble layer is added so that the application has a file system
    - Any data in a container layer are lost when the container is stopped
    - Docker uses a Copy on Write (CoW) strategy
        - So if a layer needs to read a file in a lower layer it just does so directly
        - If it needs to write to the lower layer it will copy the file from the lower layer and then change it. Given how the OvelayFS works this will override the file in the lower layer.
        - Layers are reused where possible, so if a layer is already in the local Docker storage it will just be used instead of pulling from the repo
### Docker swarm
    - A way of running the docker engine that allows for multiple nodes. 
    - Nodes come in 2 types Manager and Worker
    - It uses the Raft consensus algorithm to get the managers to agree on the status of data.
### Docker compose
    - Allows for the orchestration of multiple containers including the setting up of networking between them
    - The list of containers is described in a docker compose yaml file
    - It comes with it's own CLI commands
### Docker persistent storage
    - Docker containers can make data persist by using persistent storage mechanisms of docker. These are:
    - tmpfs mounts
        - Temporary storage that is not persistent, instead it is inside the memory of the host machine
        - Is faster than writing to disk (obviously)
        - Good for temp store of sensitive data
    - Bind mounts
        - Can be stored anywhere on the filesystem
        - Can be modified by any process
        - Usage
            - During docker run or inside the docker compose yaml
                - For `docker run` you need the `--mount` flag
    - Volumes
        - Stored in a part of the host filesystem which is managed by docker
        - Non docker processes should not edit the contents of these files
        - Usage
            - Can be created either by:
                - CLI command `docker volume create`
                - During docker run or inside the docker compose yaml
                    - For `docker run` you need the `--mount` flag
                - During service creation
### Docker networking
- Docker creates 3 networks of 3 different drivers by default: bridge, host, null
- The default network for containers is the default bridge network
- Bridge networks allow containers to talk to each other via IPs, in order to access stuff on containers that are on this network you must expose the ports using the `-p` flag mentioned earlier
- Host networks allow the container to be reached on host network (read as localhost)

____
# Lecture 2 review - Kubernetes
## Control plane architecture
- Control plane: 1+ Master nodes
    - Basically so you can have a cluster of real machines 
- Master node: Physical or VM that manages the cluster and never runs user codes or images etc
- kube-apiserver: Exposes the Kubernetes api
- etcd: Stores key - value pairs used in the configuration of Kubernetes
- kube-scheduler: selects the node on which to run newly created pods
- kube-controller-manager: runs controller processes (Node, Replication, Endpoints, Service Accounts, and Token controllers)
- Cloud controller manager:  connects the cluster to the cloud provider's API, separating components that interact with the cloud platform from components that interact with the kubernetes cluster.
##  Worker nodes architecture
- Node/Worker/Minion: Physical or VM that executes user workloads, interacts with the control plane to organise this
- kubelet: Ensures that pods are running in the node as declared by their specifications
- kube-proxy: network proxy for maintaining the rules on nodes
- pod: group of one or more containers that share resources described by a specification about how to run containers
## Kubectl
- Is the command used to interact with all things kubernetes
- Needs to be configured to connect to the control plane
    - This has to be setup when NOT using microk8s or minikube
## Nodes
- A physical or VM that runs the kubelet agent, kube-proxy, and Network pod
- Kubelet has to be installed manually (when NOT using microk8s and minikube)
- A network add-on and network pod has to be installed manually (when NOT using microk8s and minikube)
## Namespaces
- Used to create virtual clusters within a cluster
- By default #K8s (#Kubernetes) creates objects in the default namespace
- K8s components run in their own namespace called kube-system
- Namespaces cannot be nested
- **Namespaces, persistent volumes and nodes aren’t inside a namespace**
## Pods and containers
- Pods are the smallest unit you can deploy to a K8s cluster
- A pod usually has one container, but can have more than one
- **All containers in a pod share resources**
- To scale horizontally we add more of the same pod, not more containers in a pod
- There are init containers that run inside the pod until a start point has been reached, and then they are killed
- Can be created directly using kubectl, most of the time however created by other objects like Jobs,  Deployments, etc
### Phases Pods can go through
- **Pending**
    - Pod is accepted as valid by cluster and created but the containers inside the pod are not running yet
- **Running**
    - The pod is bound to a node. All containers have been created. At least one container is running, starting, or restarting
- **Succeeded**
    - All containers terminated successfully, they will not restart
- **Failed**
    - All containers terminated, at least one container terminated with an error
- **Unknown**
    - Status cannot be detected
- **CrashLoopBackoff**
    - Something is wrong with the configuration of the pod and one it's containers exited unexpectedly

- Can run stuff in pods using `kubectl exec` command


## Workload resources and controllers
- Types of workload resources
    - Deployment
    - ReplicaSet
    - DaemonSet
    - Job
    - CronJob
    - StatefulSet
- When a workload resource is submitted to K8s the control plane configures a corresponding controller
- Controllers run in control loops: infinite loops in which the controller checks the status of the resources requested from the cluster and if status is not the expected one it tried to fix the solution
### Spec and status
- Most K8s objects have two fields: *spec* and *status*
- Spec describes the desired state of the created object, written in the yaml when making the specification for the object
- Status describes the current status of the object, is retrieved when you get the yaml description of an object at runtime
### Deployments
- Type of workload that is great for running stateless applications like webservices
- In the Deployment specification the number of desired replicas and the specification of the containers you want to run are defined
- Deployment controller manages the number of replicas, by creating or deleting them as needed
- **ReplicaSets**
    - **An object whose purpose to maintain a desired number of pods based on a specification**
    - Generally not made by users, automatically made when a Deployment is created
- **HorizontalPodAutoscaler**
    - Object whose specification contains conditions for when to scale up or down the pods of a deployment
### DaemonSet
- Ensures that pods run on each node of the cluster even if nodes are added at a later time
- _A typical example of use for a DaemonSet is a process that collects log on each Node of the Cluster._
### Job
- Ensures that a specific task completes even if the pods running the task fail
- Does so by checking the status of the Pod and launching a new one if it is an error or failure
- Can run pods sequentially or in parallel
- `parallelism` attribute defines how many pods should be run in parallel
- `completions` indicates how many pods must complete for the Job to be considered done
- `backOffLimit` indicates how many pods should fail before the job itself is considered failed
### CronJob
- Same as a Job but runs on a schedule
### Garbage Collector
- Is a mechanism that is part of the kubelet that deletes objects when their parents are deleted

______
# Lecture 3 review

## Networking
### OSI Model
1. Physical
    - Transmission of raw bit streams over physical medium
2. Link
    - Node to node data transfer, may use Point to Point protocol and it's derivatives
3. Network
    - Transfer packets among nodes. Internet protocol is used to relate datagrams across network boundaries.
4. Transport
    - Transmission Control Protocol (TCP) and User Datagram protocol (UDP). TCP provides
reliable, ordered, and error-checked delivery of a stream of bytes between applications running
on hosts communicating via an IP network.
5. Session
      - Network sockets are created by network APIs and use handles that are a type of file descriptor
on Unix-like systems. They establish a long lasting communication channel.
6. Presentation
      - Purposes: character encoding, compression, decompression, encryption, decryption. Examples
protocols: MIME, SSL, TLS
7. Application
      - High level APIs: file access, resource sharing. Example protocol: WebSockets
   - Switches connect devices within the same network and typically operate at the data link layer
   - Routers connect devices across networks, they operate at the 3rd layer of OSI (Network)
   - Ip forwarding used on a machine that is connected to 2 networks, to direct messages from one network to the other network
       - Can have more than 2 networks
   - Networks can have namespaces to split a network within itself
   - Can then be connected by forwarding

## Kubernetes networking
### The Kubernetes network model imposes constraints on third party network implementation:
- Every Pod has its own IP address, Pods on the same Node can see each other by default
- Containers withing a pod share Pod IP address and can communicate with each other like processes on a machine
- Pods on a node can communicate with all pods on all nodes without NAT
    - NAT is Network Address Resolution, which is a method of mapping an IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device.
- Kubernetes agents can communicate with all pods on that node
- On hosts where a Pod(s) have been configured to share the host IP address Pods, can reach all pods on the network


- You can set rules in Kubernetes for networking via Network Policies
- In order to do this a third party networking solution must be installed, (in microk8s Calico is installed by default to do this)


### The Container Network Interface
- The Container Network Interface (CNI) specification, defines what parts of networking are the responsibility of the Container Runtime and what is the responsibility of Network Plugins
    - Container Runtime
        - Creating a new network namespace for the container before invoking any plugins
        - Determining which networks this container should belong to, and for each one, which plugins must be executed
        - Adding the container to each network by executing the corresponding plugins for each network sequentially
        - Executing the plugins in reverse order of the order they were added to the container, to disconnect the container from the networks when the container is terminating
    - Network Plugins
        - Inserting a network interface into the container network namespace
        - Attaching the other end of the veth (virtual ethernet) into a bridge on the host
        - Assigning the IP to the interface
        - Setting up the routes consistent with the IP Address Management section by invoking appropriate IPAM plugin

#### Pods routable vs non-routable
- Pods can be **non-routable (not reachable outside of cluster without special rules)** or **routable (reachable from outside the cluster without special rules)**
    - Non-routable
        - Pod IP address is not known outside the cluster
        - **Connection from Pod to outside the Cluster requires** **_SNAT_** (_Source Network Address Translation_) to change the source IP address from IP address of the Pod to the IP address of the Node the Pod is running on
        - Return packets are mapped automatically from the Node IP to the Pod IP
        - No connections from outside the cluster to the Pod are possible (unless you create Kubernetes Services or Kubernetes Ingress)
    - Routable
        - Pod IP address is known outside the cluster, so the Pod IP address must be unique in the broader network
- Pods are not permanent and are instead dynamically created and destroyed, as such so too are there IP addresses. In order to have a way to access pods that are doing a thing we use Services.
### Kubernetes Services
- Services define a logical set of pods so they can be selectable
    - With labels on Pods that makes can be queried
    - Without labels
        - This works by essentially defining an endpoint for a service that points to the exact resources for which to forward the requests
- They also specify how to access these pods
### Service Types
- Creating a service creates an endpoint so that the appropriate pods or resource can be reached via forwarding from the service
#### ClusterIP
- Exposes the Service on a cluster-internal IP address
- The Service is only reachable from within the cluster
- This is the default Service Type
- It gets created automatically when you create a NodePort or LoadBalancer Service
##### ClusterIP - Usage
- The pods behind a clusterIP service are not visible outside the cluster
- A port and targetPort are defined as part of the service, which means that other objects can access the pods by hitting the IP of the service at the port and it will forward it to a pod at the target port
- The ClusterIP service will load balance among the pods it controls
- Example use case is to allow the backend to be easily reachable from the frontend without exposing the backend to the world
#### NodePort
- Exposes the server on each Node's IP at a static port (chosen from a range 30000-32767)
- The NodePort automatically creates a ClusterIP service
- The NodePort service routes to the ClusterIP service
- Clients can reach the NodePort Service, from outside the cluster by requesting any Node's IP on the specified NodePort 
##### NodePort - Usage
    - The pods are not directly visible to the outside world instead a port is opened on every Node that corresponds to the defined Nodeport
    - when this NodePort is hit the request is forwarded to the Service at the port it has defined 
    - the service forwards the request to the appropriate pod or resource at the specified Port
- In case the pods of a service are on more than one node then the requests will be forwarded according to which node is hit or any rules setup in an ingress to deal with this
#### LoadBalancer
- Exposes the Service externally using an external load balancer (which is not part of the standard K8s installation)
- It automatically creates a ClusterIP and NodePort
- On cloud providers the LoadBalancer implementation is provided by them 
- On bare metal, you have to install a load balancer, e.g. MetaLB
- The LoadBalancer Service routes to the NodePort and ClusterIP Services it created
- The service can be accessed from outside the cluster on an External IP
- If no load balancer is installed then External IP will be pending forever
##### LoadBalancer - Usage
- A LoadBalancer service combines a ClusterIP and NodePort service plus exposing a public IP address that the service can be reached on
- This means that such a service can be accessed within the clusterIP by the service name or IP
- It can also be accessed by the assigned NodePort on the IP address of the Node
- It can also be accessed by the outside world by hitting the external IP created
### kube_proxy
- Kubernetes Services are implemented by kube_proxy, which works in 4 ways
    - userspace
    - iptables
    - ipvs
    - winuserspace
### Kubernetes Ingress
- **Ingress is a load balancer that is configured on the Application layer of the OSI Model (Layer 7)**
    - Load Balancers that operate on Layer 4 operate using TCP. They forward to the packets of a message based on the inspection of the first few packets
    - Load balancers that operate on Layer 7 operate using HTTP. They can decide where to forward requests based on the full content of the message
- **Ingresses are useful when you have more than one Service you want to expose publicly**
- Ingress is not part of the default K8s installation, an ingress controller must be installed for it to work
    - Many Ingress controllers out there, you must pick
- You also need to create an Ingress K8s object that will hold all the rules 
- Rules are set up with some specific K8s key value pairs such as paths and pathType, as well as through some annotation that describe the behaviour of the controller
### Kubernetes CoreDNS
- **Kubernetes allows the installation of CoreDNS which creates configMaps that allow the mapping of services to their IPs so that they can be reached by name from withing the cluster**
- It does so by injecting the required info into the hosts file of pods
### Kubernetes Network Policies
- Are objects that allow you to restrict how pods can communicate inside a cluster
- You can isolate pods by using this mechanism
- Network policies allow you to define Ingress and Egress rules that are applied to pods using a selector
        

___
# Lecture 4 review
## Kubernetes Storage
- In K8s volumes are attached to the lifecycle of a Pod
- Volumes are mounted to containers using another writeable layer
- If there is more than one container inside a Pod then volume will be accessible by all containers (at potentially different locations)
- Different types of volumes that determine what happens to the data of a volume when the pod no longer exists
- Volumes are directories accessible to the Pods
- There are some general rule for volumes
    - The root directory is the filesystem of the container image
    - Volumes can be mounted at specific paths within containers
    - Volumes can not be mounted onto other volumes
    - Each container in the pod must specify where to mount each volume
### Volume types
- emptyDir
    - Volume type that gets created when a Pod is attached to a Node
    - Is empty to start with, hence the name
    - When the containers crash or die the emptyDir volume remains untouched
    - When the Pod crashes or dies then the emptyDir volume gets deleted and the data is lost
    - All containers in the pod can access the volume 
    - Can be set so that the data is in memory for faster albeit temporary file storage
    - Data is actually stored on the node itself
- hostPath
    - A volume type that allows containers to access a path on the host (node) filesystem
    - hostPath makes a lot of sense for single node clusters but not very useful for multi nodes since it will be unknown on which node is the specified hostPath on
    - The data is kept after a Pod dies but the volume object dies with the pod
- nfs (Network File System)
    - Volume type that allows the use of an existing Network File System (NFS) for a Pod
    - K8s doesn't provide nfs capability you have to do that on your own
    - When a pod dies the data stays as is in the nfs but the volume object dies with the pod
    - Multiple writers at the same time


- There are also Persistent Volumes whose lifecycle is decoupled from the lifecycle of Pods
- Persistent Volumes are used by pods using Persistent Volume Claims
- Persistent Volume Claims
    - Define the capacity needed by pods
    - Define the AccessMode to the Persistent Volume: ReadWriteOnce, ReadOnlyMany, or ReadWriteMany
        - ReadWriteOnce -- the volume can be mounted as read-write by a single node  
        - ReadOnlyMany -- the volume can be mounted read-only by many nodes  
        - ReadWriteMany -- the volume can be mounted as read-write by many nodes (usually not supported)
        - *Note:* not all access modes are always supported on different providers
    - Kubernetes will try and match a PVC to a PV, if one of the same size is not found then a bigger one may be used, the extra space will remain untouched
    - PVs and PVCs are matched one to one
    - _PVs maybe created statically or dynamically_
        - **Statically**
            - The sysadmin makes the PVs ahead of time and PVCs are used to get access to them
            - Catch is there might not be an appropriate PV for a PVC
        - **Dynamically**
            - When none of the PVs already existing and available don't match the PVCs, a PV is created dynamically
            - For this to be possible **DefaultStorageClass** admission controller must exist on the API server, additionally the PVC must request a storage class and that class must exist in the cluster
            - This solves the potential problem of wasted space when all your PVs are bigger than your PVCs requested size
    - **PVs also have reclaim policies which describe what happens to the data after the relevant PVC is deleted**
        - **Retain: The PV is kept after the PVC is deleted**. 
        - Delete: The PV is deleted along with the PVC, additionally on host providers the underlying storage is also deleted, on hostPath I guess the data is deleted
    - 
## StatefulSet
- Can be used instead of a Deployment to manage stateful applications
- Manages a set of pods based on a template, like a Deployment
- Unlike a Deployment a StatefulSet makes it easy to access individual pods, **you can also specify the order of creation of pods**
- Good for highly available things where you have to specify where to replicate things
- Using StatefulSets also makes it easier to couple the existing volumes to the new Pods that replaced crashed ones
- To make a StatefulSet you first have to make a headless service, i.e. a ClusterIP service with the ClusterIP set to none
## ConfigMaps
- Are objects that are used to store the values needed for running an application like URLs, passwords etc
- They make it easy to have different environments like dev, test, prod, etc
- There are ConfigMaps and Secrets
- ConfigMaps are for non confidential data, Secrets are for confidential data
- Values from both can be read in 4 ways
    - Environment variables that are set in the spec of a container
    - Command line arguments passed to the container inside a pod
    - Read-only config files in a volume that can be read by a pod
    - Container code that calls the K8s API to get data from the ConfigMap object
- Secrets are not encrypted but are base64 encoded
- Secrets have different types for different uses. Opaque is the default or arbitrary user data
- Secrets are also used for securing network components with TLS in K8s, there is a secret type called kubernetes.io/tls
## Cert-manager
- The creation of certificates can be setup to be done automatically with cert-manager
- It introduces some customer K8s object types
    - Issuer: Namespaced resource that can issue certificates for the namespace it is in
    - ClusterIssuer: Same as Issuer but for all namespaces
    - Certificate: Namespaced object that contains reference to the Issuer or ClusterIssuer that will be honoring the certificate request, contains the details of the request
    - CertificateRequest: Namespaced resource that is used to request certificates from the Issuer or ClusterIssuer, contains the base64 encoded PEM encoded request

____

# Lecture 5 review
## Rolling updates
    - A mechanism that can be specified in a deployment to ensure that when an update occurs the old version stays available while the new version is being created
    - Each that gets created from a deployment gets a name that inlcudes the hashed values of the PodTemplate and ReplicaSet that it is associated with it
    - This way if the deployment is changed then there is no overlap and the old pods can be deleted
    - Deployments can have maxUnavailable and maxSurge defined that determine the behaviour of the rollout
    - maxUnavailable will determine how many pods can be taken down while creating the new ones, default is 25%
    - maxSurge is how many extra pods can be created as part of the scaling up process
## Canary deployments
- Are used to test a new release to a subset of users
- Name comes from a canary in a coal mine
- To make a canary deployment you need a service that can route traffic to pods using old code or pods using new code
- Usually the routing is done based on the labels of the pods
#### Usage
- Create a deployment that has your stable version, with labels and environment vars on the pod that indicate this
- Create another deployment with the new version, with labels and environment vars on the pod that indicate this
- Everything will be identical in these two deployments other than the version stuff mentioned above
- Create a NodePort Service that selects the pods it "controls" based on the application or some other field but not the version
- Scaling down the stable version will scale add pods from the new version based of course on the replicas in the deployments 
- When happy with the testing scaling up the new version and deleting the old version will cause the new version to be the only deployment the service routes traffic too
## Helm
- K8s package manager
- Introduces a thing called a "Helm chart" to manage the orchestration of all the components of an application deployed to K8s
- Charts can be stored in repositories like HCL 
- Helm has its own CLI interface
- Charts have template yaml files for all the necessary components, a Chart.yaml file and a values.yaml file
    - Chart.yaml defined whether a chart is an application or a library, libraries cannot be deployed standalone
    - values.yaml values defined in yaml syntax to be used in the templates to make things a little more generic
- A chart can include another chart.
- Helm charts can have hooks to run custom code, they can be used to start up a Pod or a Job.
# Lecture 6 review
## Network policies
- Network policies allow the definition of the inbound and outbound rules of traffic for K8s objects
- Egress = outbound from objects
- Ingress = inbound to objects
- A network plugin must be added to your K8s cluster for the rules to take effect
- Policies are additive, i.e. if you have an empty policy and it is affecting a pod then it will have no permissions
- You select the objects using the podSelector or namespaceSelector to which the policies apply for ingress or egress
## Role Based Access Control
- Roles and RoleBindings can be defined to control access to the K8s API
- Roles can be bound to cluster user, group, or ServiceAccount
    - Service accounts are kubernetes accounts that can manage objects based on the namespaces they are allowed to access
    - *Note:* Without some trickery cluster users are not same as OS users
- **Roles and RoleBindings are namespaced, the non namespaced equivalent is ClusterRole and ClusterRoleBindings**
- **Permissions in the Roles are additive**
- They are set by defining resources and then for the resources what you are allowed to do with them
- You can separate the resources so that they have different permissions

## Affinity and Anti-affinity
- Usually pods are placed on a node based on a scheduler that tries to make sure resources are used in an efficient way
- This can be affected by using specific labels
- Might want to do this for many reasons
    - Ensure that sensitive pods are not on the same node for resilience
    - Ensure that pods that require specific hardware are placed where they need to be
    - Ensure that pods that have a lot of communication are on the same machine to reduce latency
- This can be done with the nodeSelector key in the YAML, when you want to specify the node that a pod should be put on based on the nodes attributes
- This can be done with the podAffinity when you want to specify that the pod should be next to another pod i.e. on the same node as another type of pod
## Taints
- Are ways to label nodes such that they reject all pods that don't have that label as a toleration
    - Essentially you mark the node as unable to accept anything apart from things with that label as a toleration
- Tolerations
    - Are the labels that correspond to a taint that will allow the scheduler to place a pod on the tainted Node
## Pod disruptions
- Pods can be disrupted for a number of reasons, either involuntary such as crashes, fires, cyclones, etc or voluntary such as scaling or deleting a pod or etc
- To limit the effect of voluntary disruptions when there might already be a limited number of resources available there is something called a PodDisruptionBudget
- A PodDisruptionBudget determines when a Pod can be disrupted voluntarily either with minAvailable or maxUnavailable but not both
