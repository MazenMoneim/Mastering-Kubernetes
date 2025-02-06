<h1 align="center">
    <img src="https://readme-typing-svg.herokuapp.com/?font=Righteous&size=35&center=true&vCenter=true&width=500&height=70&duration=4000&lines=Deed+Dive+on+Kubernetes!;" />
</h1>



<h3 align="center">A Deep Dive into Kubernetes Architecture and Components</h3>
 
<div align="center">
    <img src="https://skillicons.dev/icons?i=kubernetes" width="150" height="120" /><br>
</div>


<hr/>


Kubernetes (K8S) is utilized for hosting applications in a containerized and automated manner. The architecture is divided into Master Nodes and Worker Nodes, each with specific responsibilities:

- **Master Nodes**: Manage, plan, schedule, and monitor the nodes.
- **Worker Nodes**: Host applications as containers.

Key components within the nodes include:

- **ETCD**: Stores cluster information.
- **Kube-Scheduler**: Schedules containers on nodes.
- **Kubelet**: Listens for instructions from the API server and manages containers on the node.
- **Kube-proxy**: Enables communication between services within the cluster.

**Containerd** serves as the container runtime interface, and interaction with this daemon can be done using tools like `ctr`, `nerdctl`, or `crictl` from a Kubernetes perspective.

Kubernetes interacts with Docker through a middleware utility called **dockershim**. However, Kubernetes now relies on Containerd instead of Docker, with dockershim being used primarily for debugging purposes.

<br/>

<div align="center">
  <img src="https://github.com/user-attachments/assets/57d55129-cd1f-49b4-9f7c-31d583a00d56" width="600"/>
</div>

<br/>
<hr/>
<hr/>


<html>
<head>
    
</head>
<body>
    <h1>🐳 ETCD</h1>
    <p>
        ETCD is a distributed, reliable key-value store. It operates as a service similar to other services in Linux. 
        To set it up, download the binaries, extract them, and run the service. ETCD runs on port 2379.
    </p>
    <h2>Role in Kubernetes</h2>
    <p>
        In Kubernetes, ETCD stores critical information about the cluster, including nodes, pods, configurations, 
        secrets, accounts, roles, and role bindings. All information retrieved using the <code>kubectl get</code> 
        command comes from the ETCD server. Any changes made to the cluster, such as adding nodes or deploying pods 
        and replica sets, are updated in the ETCD server.
    </p>
    <h2>Deployment</h2>
    <h3>Manual Deployment</h3>
    <p>If deploying your cluster from scratch, download the ETCD binaries and run the service.</p>
    <h3>Using kubeadm</h3>
    <p>ETCD is deployed in the cluster as a pod in the kube-system namespace.</p>
    <h2>Listing Keys</h2>
    <p>To list all keys stored in ETCD, use the following command:</p>
    <pre><code>kubectl exec etcd-master -n kube-system -- etcdctl get / --prefix --keys-only</code></pre>
    <h2>ETCDCTL Utility</h2>
    <p>
        ETCDCTL is the CLI tool used to interact with ETCD. It supports two API versions: Version 2 and Version 3. 
        By default, it uses Version 2, and each version has different sets of commands.
    </p>
    <h3>Version 2 Commands:</h3>
    <ul>
        <li><code>etcdctl backup</code></li>
        <li><code>etcdctl cluster-health</code></li>
        <li><code>etcdctl mk</code></li>
        <li><code>etcdctl mkdir</code></li>
        <li><code>etcdctl set</code></li>
    </ul>
    <h3>Version 3 Commands:</h3>
    <ul>
        <li><code>etcdctl snapshot save</code></li>
        <li><code>etcdctl endpoint health</code></li>
        <li><code>etcdctl get</code></li>
        <li><code>etcdctl put</code></li>
    </ul>
    <h2>Setting API Version</h2>
    <p>To set the API version, use the following environment variable:</p>
    <pre><code>export ETCDCTL_API=3</code></pre>
    <p>
        When the API version is not set, it defaults to Version 2. Version 3 commands will not work unless the 
        API version is explicitly set to 3, and vice versa.
    </p>
    <h2>Authentication</h2>
    <p>
        To authenticate ETCDCTL to the ETCD API Server, specify the path to the certificate files. These files 
        are available in the etcd-master at the following paths:
    </p>
    <ul>
        <li><code>--cacert /etc/kubernetes/pki/etcd/ca.crt</code></li>
        <li><code>--cert /etc/kubernetes/pki/etcd/server.crt</code></li>
        <li><code>--key /etc/kubernetes/pki/etcd/server.key</code></li>
    </ul>
    <h2>Example Command</h2>
    <p>Here is an example command that sets the API version and specifies the certificate files:</p>
    <pre><code>kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key"</code></pre>
</body>
</html>

<br/>
<hr/>
<hr/>
<html>
<head>
</head>
<body>
    <h1>🐳 Kube-API-Server</h1>
    <p>
        The <strong>Kube-API-Server</strong> is the primary management component in Kubernetes. It serves as the central 
        communication hub for all components within the cluster.
    </p>
    <ul>
        <li>
            When you run <code>kubectl</code> commands, the <code>kubectl</code> utility interacts with the Kube-API-Server.
        </li>
        <li>
            The Kube-API-Server first authenticates and validates the incoming request. It then retrieves the required 
            data from the ETCD cluster and responds with the requested information.
        </li>
        <li>
            You don't necessarily need to use the <code>kubectl</code> utility; you can directly interact with the API 
            by sending POST requests.
        </li>
    </ul>
    <p>
        The Kube-API-Server is the only component that communicates directly with the ETCD cluster.
    </p>
</body>
</html>
<br/>
<hr/>
<hr/>
<html>
<head>
</head>
<body>
    <h1>🐳 Kube-Controller Manager</h1>
    <p>
        The <strong>Kube-Controller Manager</strong> is responsible for monitoring and taking necessary actions on containers within the Kubernetes cluster.
    </p>
    <ul>
        <li>
            It continuously monitors the status of the system components and takes necessary actions to remediate any issues.
        </li>
        <li>
            In Kubernetes terms, a controller is a process that continuously monitors the state of various components within the system and works towards bringing the whole system to the desired functioning state.
        </li>
    </ul>
    <h2>Types of Controllers</h2>
    <ul>
        <li>
            <strong>Node Controller:</strong> Responsible for monitoring the status of the nodes and taking necessary actions to keep the application running. The Node Controller checks the status of the nodes every 5 seconds (heartbeat).
            <ul>
                <li>
                    The node monitor period grace is the duration that the Kubernetes Controller Manager waits before marking a node as unhealthy if it hasn't received a status update from the node. By default, this period is set to 40 seconds.
                </li>
                <li>
                    The Node Controller gives the node 5 minutes to return before the workload on this node is evacuated to another node.
                </li>
            </ul>
        </li>
        <li>
            <strong>Replication Controller:</strong> Responsible for monitoring the status of replica sets and ensuring that the desired number of pods are available at all times within the set. If a pod dies, it creates another one.
        </li>
        <li>
            There are many other controllers like the Deployment Controller, PV-Binder-Controller, Endpoint Controller, and Namespace-Controller.
        </li>
    </ul>
    <p>
        All controllers are packaged into a single process known as the Kubernetes Controller Manager.
    </p>
</body>
</html>
<br/>
<hr/>
<hr/>
<!DOCTYPE html>
<html>
<head>
</head>
<body>
    <h1>🐳 Kube-Scheduler</h1>
    <p>
        The <strong>Kube-Scheduler</strong> is responsible for scheduling pods on nodes within the Kubernetes cluster. It is tasked with determining which pod goes on which node, although it does not actually place the pod on the nodes. This task is performed by the Kubelet, which creates the pod on the node.
    </p>
    <h2>Scheduling Phases</h2>
    <ul>
        <li>
            <strong>Filter Nodes:</strong> The scheduler filters out nodes that do not have sufficient CPU and memory resources as requested by the pod.
        </li>
        <li>
            <strong>Rank Nodes:</strong> The scheduler ranks the remaining nodes to identify the best fit for the pod using a priority function. It calculates the amount of resources that would be free on the nodes after placing the pods on them.
        </li>
    </ul>
</body>
</html>
<br/>
<hr/>
<hr/>
<html>
<head>
</head>
<body>
    <h1>🐳 Kubelet</h1>
    <p>
        The <strong>Kubelet</strong> is responsible for registering the node with the Kubernetes cluster. It plays a crucial role in managing and maintaining the state of pods and containers on each node.
    </p>
    <ul>
        <li>
            When the Kubelet receives instructions to manage a container or a pod on the node, it requests the container runtime engine (such as Docker or containerd) to pull the required image and run an instance.
        </li>
        <li>
            It continuously monitors the state of the pod and its containers and reports this status to the Kube-API-Server.
        </li>
        <li>
            The Kubelet must be downloaded and installed on each node, even when using the kubeadm tools.
        </li>
    </ul>
    <p>
        To set up the Kubelet, download the binaries, extract them, and run the Kubelet as a service on each node.
    </p>
</body>
</html>
<br/>
<hr/>
<hr/>

<html>
<head>
</head>
<body>
    <h1>🐳 Kube-Proxy</h1>
    <p>
        The <strong>Kube-Proxy</strong> is a process that runs on each node within the Kubernetes cluster. Its primary responsibility is to manage network connectivity and routing for services.
    </p>
    <ul>
        <li>
            Kube-Proxy monitors the cluster for new services. Whenever a new service is created, it configures the appropriate rules on each node to ensure traffic is forwarded to the backend pods.
        </li>
        <li>
            One method Kube-Proxy uses to achieve this is by creating <code>IPTABLES</code> rules.
        </li>
    </ul>
</body>
</html>




<br/>
<hr/>
<hr/>


<html>
<head>
</head>
<body>
    <h1>🐳 Kubernetes Pods</h1>
    <p>
        Containers are encapsulated into a Kubernetes object known as Pods. If you need to scale your application, 
        you would create additional Pods. In a multi-container Pod, the containers are not of the same kind; 
        these are called sidecar containers, responsible for performing supporting tasks for the main application.
    </p>
    <p>
        Pods by default share the same storage, network namespace, and fate (they are created together and destroyed together).
    </p>
    <p>
        When you run <code>kubectl run nginx --image nginx</code>, it will pull the nginx image from Docker Hub.
    </p>
    <p>
        You can configure Kubernetes to pull images from Docker Hub or a private repository within your organization.
    </p>
    <h2>Useful Commands</h2>
    <ul>
        <li><code>kubectl create -f file-name.yml</code></li>
        <li><code>kubectl get pods</code></li>
        <li><code>kubectl describe pod pod-name</code></li>
    </ul>
</body>
</html>




<br/>
<hr/>
<hr/>



<html>
<head>
</head>
<body>
    <h1>🐳 Replica Set</h1>
    <p>
        The <strong>Replica Set</strong> is the newer technology that replaces the older Replication-Controller. It manages the replica sets and ensures that the desired number of pods are running within the Kubernetes cluster.
    </p>
    <h2>Replica Set File Definition</h2>
    <p>
        One key difference between the Replication Controller and the Replica Set is the use of the selector. The selector allows managing pods with this selector even if the pods exist outside of the replica set and were created before the replica set.
    </p>
    <p>
        The Replica Set is a process that monitors the state of pods. It identifies which pods to monitor in the cluster based on the selector.
    </p>
<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/7c65f51c-5554-4d56-84db-e20c445805c4" width="800"/>
</div>
<br/>
    <h2>Scaling Pods</h2>
    <p>
        To scale in or out the number of pods in the replica set, change it in the yaml file and use the following command:
    </p>
    <pre><code>kubectl replace -f file-name.yml</code></pre>
    
    



<h2>Editing Existing Objects</h2>
<p>
    If you need to edit an existing running object in Kubernetes, use the following command:
</p>
  <pre><code>kubectl edit object-type object-name</code></pre>
<p>
   This approach is useful when you don't have the running object's file and need to update it. After editing, delete the pods, and Kubernetes will recreate them with the updated configuration.
</p>
</body>
</html>




<br/>
<hr/>
<hr/>


<!DOCTYPE html>
<html>
<head>
</head>
<body>
    <h1>🐳 Deployments</h1>
    <p>
        Kubernetes <strong>Deployments</strong> provide powerful capabilities, such as rolling updates, rollbacks, and upgrades, ensuring your applications run smoothly and stay up-to-date.
    </p>
    <p>
        Deployments work in conjunction with Replica Sets, forming a layered approach. When you deploy a Deployment, it automatically manages the associated Replica Set. You can verify this by running the <code>kubectl get replicaset</code> command, which will display the Replica Set created by the Deployment.
    </p>
    <h2>Creating Templates</h2>
    <p>
        For the exam, please use the following commands to create templates for pods or deployments:
    </p>
    <ul>
        <li><code>kubectl run nginx --image=nginx --dry-run=client -o yaml</code></li>
        <li><code>kubectl create deployment --image=nginx nginx --dry-run=client -o yaml</code></li>
    </ul>
</body>
</html>

<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/2243b921-3b68-42b0-8b83-75aae97cb877" width="800"/>
</div>
<br/>




<br/>
<hr/>
<hr/>

<h1>🐳 DaemonSets</h1>
    
<h2>Overview</h2>
<p>
    A <strong>DaemonSet</strong> ensures that a copy of a Pod runs on each node in the Kubernetes cluster. DaemonSets are particularly useful for deploying system-level applications that need to run on all nodes.
</p>
    
<h2>Functionality</h2>
<ul>
    <li>Runs one copy of your Pod on each node in your cluster.</li>
    <li>Automatically adds a replica of the Pod to any new node added to the cluster.</li>
    <li>Ensures that a copy of the Pod is always present on all nodes in the cluster.</li>
</ul>
    
<h2>Examples</h2>
<p>Common use cases for DaemonSets include:</p>
<ul>
    <li>Monitoring solutions</li>
     <li>Log viewers</li>
    <li>Network solutions</li>
</ul>
    
<h2>Node Affinity</h2>
<p>
     DaemonSets use node-affinity rules to schedule Pods on specific nodes.
</p>

<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/e83c461e-dac8-4f8c-bb73-95d9aa4b0725" width="800"/>
</div>
<br/>


<br/>
<hr/>
<hr/>



<h1>🐳 Static Pods</h1>
    
<h2>Overview</h2>
<p>
    The kubelet relies on the kube-apiserver for instructions on creating nodes. But what if there is no kube-apiserver? What if there is no master or cluster at all? In such cases, the kubelet can manage its node independently. The kubelet knows how to create pods, even without a kube-apiserver.
</p>

<h2>Configuration</h2>
<p>
    You can configure the kubelet to read the pod definition files from a designated directory, such as <code>/etc/kubernetes/manifests</code>. The kubelet periodically checks this directory for files, reads them, creates the pods, and monitors them. If a pod crashes, the kubelet will recreate it. Removing the definition file from the directory will automatically delete the pod.
</p>
<p>
    These pods, created by the kubelet on its own without intervention from the API server, are called <strong>static pods</strong>. You cannot create deployments or other Kubernetes objects with static pods; only pods are supported.
</p>
<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/0380e4be-9cb7-4dd0-9c99-ebf9d3b655b9" width="800"/>
</div>
<br/>


<h2>Behavior</h2>
<p>
    If there is a Kubernetes cluster with static pods, running <code>kubectl get pods</code> will show a read-only mirror of the pod. You can describe it, but you cannot delete or edit it.
</p>

<h2>Usage of Static Pods</h2>
<p>
    Static pods are used to deploy master components. This is how tools like kubeadm set up clusters. Static pods appear in the cluster with the node name appended.
</p>

<h2>Managing Static Pods</h2>
<p>
    The kubelet runs on the controller node as a service. You can check for options of the service using <code>ps -aux</code> or by searching for static pod options in the config file.
</p>

<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/0400e83f-ac70-419b-b21e-5573f3dfd68c" width="800"/>
</div>
<br/>
<h2>Deleting a Static Pod</h2>
<ol>
    <li>
         Identify the node where the static pod is created:
         <pre><code>kubectl get pods --all-namespaces -o wide | grep static-greenbox</code></pre>
    </li>
    <li>
         SSH to the node and identify the path configured for static pods in the kubelet configuration file:
         <pre><code>ssh node01
ps -ef | grep /usr/bin/kubelet
grep -i staticPod /var/lib/kubelet/config.yaml</code></pre>
        In this example, the staticPodPath is <code>/etc/just-to-mess-with-you</code>.
    </li>
    <li>
         Navigate to this directory and delete the YAML file:
         <pre><code>rm -rf /etc/just-to-mess-with-you/greenbox.yaml</code></pre>
     </li>
    <li>
        Exit the node and check if the static-greenbox pod has been deleted:
        <pre><code>kubectl get pods --all-namespaces -o wide | grep static-greenbox</code></pre>
    </li>
</ol>

<h2>Scheduling Queue and Priorities</h2>
<p>
     When pods are created, they wait in a scheduling queue until they are scheduled. This queue has priorities, which can be set in the pod YAML file. First, you need to create a priority class.
</p>




<br/>
<hr/>
<hr/>





<!DOCTYPE html>
<html>
<head>
</head>
<body>
    <h1 >🐳 Services</h1>
    <p>
        Kubernetes <strong>Services</strong> provide a virtual IP for stable communication between components in an application. When a pod is destroyed and recreated, its IP address changes. To ensure consistent communication, Kubernetes creates an object called a service.
    </p>
    <h2>Types of Services</h2>
    <ul>
        <li>
            <strong>NodePort Service:</strong> This service listens to a port on the node and forwards requests to the pods. It provides external access to the application but is not recommended for production environments.
            <ul>
                <li>
                    Port on the pod (target port)
                </li>
                <li>
                    Port on the service (port) with its own IP address, called the cluster IP
                </li>
                <li>
                    Port on the node, which should be within the range 30,000 to 32,767
                </li>
            </ul>
            <p>
                Labels and selectors are used to link services to the targeted pods. Each node in the cluster exposes the NodePort, allowing access to the pods from any node using its IP.
            </p>
            <br/>
            <div align="center">
                <img src="https://github.com/user-attachments/assets/e26594ac-c592-4d6d-9330-899125810906" width="600"/>
            </div>
            <br/>
        </li>
        <li>
            <strong>Cluster IP Service:</strong> This service creates a virtual IP inside the cluster, enabling communication between services such as frontend, backend, and database servers.
        </li>
        <li>
            <strong>LoadBalancer Service:</strong> This service provisions a load balancer for applications in cloud providers, distributing load across different web servers. Even if your apps are deployed on just two nodes, they can be accessed using all nodes in the cluster. To achieve a single URL for end users, a cloud-native load balancer is recommended.
        </li>
    </ul>
    <br/>
    <div align="center">
        <img src="https://github.com/user-attachments/assets/02d9a3b2-677a-4085-9228-877c5a8e7809" width="700"/>
    </div>
    <br/>
    <h2>Default Service</h2>
    <p>
        The default service created when Kubernetes is launched is:
    </p>
    <pre><code>kubernetes   ClusterIP   10.43.0.1    &lt;none&gt;        443/TCP   5m17s</code></pre>
</body>
</html>


<br/>
<hr/>
<hr/>




<!DOCTYPE html>
<html>
<head>
</head>
<body>
    <h1>🐳 Namespaces</h1>
    <p>
        <strong>Namespaces</strong> are an isolation technique used to isolate projects from each other within a Kubernetes cluster.
    </p>
    <h2>Types of Namespaces</h2>
    <ul>
        <li>Default</li>
        <li>Kube-system</li>
        <li>Kube-public</li>
    </ul>
    <h2>Accessing Services Across Namespaces</h2>
    <p>
        When a web application needs to access a database in another namespace, use the following format:
    </p>
    <pre><code>Mysql.connect("db-service.dev.svc.cluster.local")</code></pre>
    <p>
        Format: <code>Service-name.Namespace.Service.Domain-name-of-the-cluster</code>
    </p>
    <h2>Creating a Namespace</h2>
    <p>
        To create a namespace, use the following command:
    </p>
    <pre><code>kubectl create namespace dev</code></pre>
    <h2>Setting a Default Namespace</h2>
    <p>
        If you need to work in a specific namespace permanently, switch to it and run this command:
    </p>
    <pre><code>kubectl config set-context $(kubectl config current-context) --namespace=dev</code></pre>
    <h2>Resource Quotas</h2>
    <p>To limit resources in a namespace, create a resource quota using the following configuration:</p>
    <pre><code>
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
    </code></pre>
    <h2>Switching Namespaces</h2>
    <p>
        To switch the namespace, use the following command:
    </p>
    <pre><code>kubectl config set-context --current --namespace=K21</code></pre>
</body>
</html>

<br/>
<hr/>
<hr/>




<html>
<head>
</head>
<body>
    <h1>🐳 Imperative and Declarative Approaches</h1>
    <h2>Imperative Approach</h2>
    <p>
        The imperative approach involves issuing commands to achieve a desired state. This approach is akin to giving direct instructions on what to do.
    </p>
    <p><strong>Example:</strong> Running commands like <code>kubectl run</code> or <code>kubectl create</code> to create resources.</p>
    <h3>Pros:</h3>
    <ul>
        <li>Simple and straightforward for quick tasks.</li>
        <li>Allows for fine-grained control and immediate feedback.</li>
    </ul>
    <h3>Cons:</h3>
    <ul>
        <li>Can become cumbersome and error-prone for complex environments.</li>
        <li>Not easily repeatable or version-controlled.</li>
    </ul>

<h2>Declarative Approach</h2>
<p>
      The declarative approach involves specifying the desired state of the system, and the system is responsible for achieving that state. This approach is akin to declaring what the end result should look like, rather than how to achieve it.
</p>
<p><strong>Example:</strong> Using YAML files to define Kubernetes resources and applying them with <code>kubectl apply -f <file>.yaml</code>.</p>
<h3>Pros:</h3>
<ul>
    <li>Easier to manage and maintain complex configurations.</li>
    <li>Changes are version-controlled and more predictable.</li>
    <li>Better suited for automated workflows and continuous integration/continuous deployment (CI/CD) pipelines.</li>
</ul>
<h3>Cons:</h3>
<ul>
    <li>Initial setup and learning curve might be higher.</li>
    <li>Requires tools and infrastructure to manage the configuration files.</li>
</ul>
</body>
</html>




<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/28822f37-0514-49a6-b457-8824627910c6" width="700"/>
</div>
<br/>


<div align="center">
    <img src="https://github.com/user-attachments/assets/016e6ea7-0796-41c1-979e-389ebd129b4c" width="700"/>
</div>
<br/>


<div align="center">
    <img src="https://github.com/user-attachments/assets/f13734dc-4caa-43ab-b733-1441844552f7" width="700"/>
</div>
<br/>





<br/>
<hr/>
<hr/>






<h1>🐳 Kubernetes Scheduler, Labels, Taints, and Selectors</h1> 

<h2>Kubernetes Scheduler</h2>
 <p>
       The Kubernetes <strong>scheduler</strong> is responsible for placing Pods onto Nodes. It watches for newly created Pods that have no Node assigned. For each Pod, it finds the best Node to run it on.
 </p>
<h3>Default Scheduler</h3>
<p>
    Kubernetes comes with a default scheduler called <code>kube-scheduler</code>. It selects an optimal node based on various factors like resource requirements, hardware/software constraints, and affinity/anti-affinity specifications.
</p>
<h3>Steps</h3>
<p>
       The scheduler performs two main steps:
 </p>
  <ul>
      <li><strong>Filtering:</strong> Finds Nodes where it's feasible to schedule the Pod.</li>
      <li><strong>Scoring:</strong> Ranks these Nodes to pick the best one.</li>
 </ul>
<h3>Custom Schedulers</h3>
<p>
      You can implement your own scheduler if the default one doesn't meet your needs. Multiple schedulers can run simultaneously.
 </p>
 <p>
     If the Pod status is pending, there might be a problem scheduling the Pod on a specific Node. In such cases, you can manually specify the node using <code>nodeName: name-of-the-node</code> in the spec field.
 </p>

<h2>Labels and Selectors</h2>
<p>
    Labels and selectors are mechanisms used to bind resources together in Kubernetes.
</p>
<p>
       <strong>Example:</strong> You can get Pods with <code>--selector app=App1</code>.
</p>

<h2>Taints and Tolerations</h2>
 <p>
  <strong>Taints on Nodes:</strong> Taints prevent unwanted Pods from landing on the node unless the Pod has a matching toleration.
</p>
 <p>
   Taints and tolerations don't tell the Pod to go to a particular node; they tell the node to only accept Pods with certain tolerations.
</p>
 <p>
      If you need to restrict a Pod from landing on a particular node, use the concept of node and pod affinity.
</p>
<p>
      To see the taints, use this command:
</p>
<pre><code>kubectl describe node kubemaster | grep Taint</code></pre>
 <p>
     To delete a taint from a node, use this command:
</p>
<pre><code>kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-</code></pre>
<br/>
 <p>
     Pod Definition with Toleration:
</p>
<pre><code>apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
    
</code></pre>

<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/ac1d6e64-1e90-4388-9642-0f1f2a6a58ef" width="800"/>
</div>
<br/>

<h2>Node Selectors</h2>
<p>
      Node selectors are used to host a Pod on a specific node.
</p>
<p>
    Limitations exist when using node selectors and labels, such as needing to host your Pod on a large or medium node and not a small node. For this, you use node affinity.
</p>


<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/23bd50a3-0f86-4b35-8868-21f5a5b4c543" width="800"/>
</div>
<br/>


<h2>Node Affinity</h2>
 <p>
    Node affinity ensures that Pods are hosted on specific nodes based on criteria.
 </p>
<p>
     The <code>Exists</code> operator doesn't need a value.
</p>
<p>
    In fact, you often use a combination of node affinity and taints/tolerations to ensure your node hosts your Pods only.
</p>




<br/>
<hr/>
<hr/>

<h1>🐳 Resource Requests</h1>
<p>
     Resource requests are a way to specify the minimum amount of CPU and memory resources that a container in a Pod should have. Here’s a bit more detail:
</p>

<h2>Resource Requests</h2>
<h3>Purpose</h3>
<p>
     Resource requests ensure that a container has the minimum amount of resources it needs to run efficiently. They are used by the Kubernetes scheduler to decide which node to place the Pod on.
</p>
<h3>CPU Requests</h3>
<p>
    The minimum amount of CPU a container requires. It is measured in CPU units. One CPU, in Kubernetes, is equivalent to 1 vCPU/Core for cloud providers, 1 hyperthread on bare-metal Intel processors, or 1 AWS vCPU.
</p>
<h3>Memory Requests</h3>
<p>
    The minimum amount of memory a container needs, measured in bytes (MiB, GiB).
</p>

<h2>How to Define Resource Requests</h2>
<p>
     Resource requests are defined in the Pod or container specification using the <code>resources</code> field in a YAML file. Here’s an example:
</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
</code></pre>
<p>
     In this example:
</p>
<ul>
    <li>The container requests a minimum of 64 MiB of memory.</li>
    <li>The container requests a minimum of 250 milliCPUs (0.25 CPU).</li>
</ul>

<h2>Importance of Resource Requests</h2>
<h3>Scheduling</h3>
<p>
    Kubernetes uses resource requests to schedule Pods onto nodes that have enough available resources.
</p>
<h3>QoS Classes</h3>
<p>
     Pods are classified into different Quality of Service (QoS) classes based on their resource requests and limits. This classification affects the scheduling and eviction behavior of the Pods.
</p>
<p>
     By default, in Kubernetes, there are no restrictions on using CPU or memory. When you use requests, you allocate the requests for the Pod, even if the Pod doesn't really need it or consume it.
</p>


<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/ed289db4-319e-41d9-98aa-2b00a820523f" width="800"/>
</div>
<br/>


<br/>
<hr/>
<hr/>

<h1>🐳 LimitRange</h1>
    
<h2>Purpose</h2>
<p>
     A <strong>LimitRange</strong> sets default values for resource requests and limits if they are not specified by the user. It ensures that all pods and containers have defined resource boundaries, promoting better resource utilization and avoiding resource contention.
</p>
    
<h2>Scope</h2>
<p>
    Applied at the namespace level. Each namespace can have its own LimitRange.
</p>
    
<h2>Types of Constraints</h2>
<ul>
     <li><strong>Minimum and Maximum Requests:</strong> Specifies the minimum and maximum CPU and memory that a container can request.</li>
     <li><strong>Default Requests and Limits:</strong> Sets default values for CPU and memory requests and limits if they are not specified.</li>
     <li><strong>Resource Limits:</strong> Ensures that no container can exceed these limits for CPU and memory.</li>
</ul>
    
<h2>Example LimitRange YAML</h2>
<p>Here's an example of how to define a LimitRange in a YAML file:</p>
<pre><code>
apiVersion: v1
kind: LimitRange
metadata:
  name: example-limitrange
  namespace: example-namespace
spec:
  limits:
  - max:
      cpu: "2"
      memory: "1Gi"
    min:
      cpu: "200m"
      memory: "100Mi"
    default:
      cpu: "500m"
      memory: "200Mi"
    defaultRequest:
      cpu: "300m"
      memory: "100Mi"
    type: Container
    </code></pre>
    
<p>In this example:</p>
<ul>
    <li><strong>max:</strong> The maximum CPU is set to 2 CPUs and memory to 1 GiB.</li>
    <li><strong>min:</strong> The minimum CPU is set to 200 milliCPUs and memory to 100 MiB.</li>
    <li><strong>default:</strong> Sets the default CPU request to 500 milliCPUs and memory to 200 MiB.</li>
    <li><strong>defaultRequest:</strong> Sets the default CPU request to 300 milliCPUs and memory to 100 MiB.</li>
</ul>
    
<h2>Importance of LimitRange</h2>
<ul>
    <li><strong>Resource Efficiency:</strong> Helps in efficient utilization of resources by ensuring that every pod/container has an appropriate amount of resources allocated.</li>
    <li><strong>Stability:</strong> Prevents a single pod/container from consuming excessive resources, which can impact the performance of other workloads in the namespace.</li>
    <li><strong>Consistency:</strong> Provides a consistent environment by setting default values for resource requests and limits.</li>
</ul>
    
<h2>Usage</h2>
<p>To create a LimitRange, use the following command:</p>
<pre><code>kubectl create -f &lt;filename&gt;.yaml</code></pre>
<p>To view LimitRanges in a namespace, use:</p>
<pre><code>kubectl get limitrange -n &lt;namespace&gt;</code></pre>
    
<p>
     By using LimitRange, you can ensure that your Kubernetes clusters run smoothly, with balanced resource allocation and utilization.
</p>




<br/>
<hr/>
<hr/>





<h1>🐳 Monitors and Logging in Kubernetes</h1>
<p>
    Monitoring involves tracking node-level metrics such as the number of nodes in the cluster, node health, CPU, memory, network, and disk utilization.
</p>
<h3>Popular Monitoring Solutions</h3>
<p>
    One of the most popular solutions for monitoring a Kubernetes cluster is the <strong>metrics server</strong>. The metrics server is an in-memory solution that does not store metrics on disk. If you need to monitor historical data, you will need an advanced solution.
</p>
<h3>cAdvisor</h3>
<p>
    The Kubelet also contains a subcomponent known as <strong>cAdvisor</strong> (Container Advisor). cAdvisor is responsible for retrieving performance metrics from pods and exposing them through the Kubelet API to make the metrics available to the metrics server.
</p>
<h3>Deployment</h3>
<p>
    Download the metrics server and deploy it as a deployment. Use the following commands to view metrics:
</p>
<pre><code>kubectl top node</code></pre>
<pre><code>kubectl top pod</code></pre>

<h2>Logging</h2>
<p>
    To see the logs of a pod, use the following command:
</p>
<pre><code>kubectl logs -f pod-name</code></pre>



<br/>
<hr/>
<hr/>










<h1>🐳 Rollout and Versioning</h1>
<h2>Rollout</h2>
<p>Rollout refers to the process of deploying updates to your applications.</p>
<ul>
<li>When you deploy your application, it creates revision 1 of the app.</li>
<li>When you update the app, it creates revision 2 of the app.</li>
<li>These revisions help us to roll back to a previous version of deployment if necessary.</li>
</ul>
<p>To see the rollout status, run the following command:</p>
<pre><code>kubectl rollout status deployment/myapp-deployment</code></pre>
<p>To see the revision history, run the following command:</p>
<pre><code>kubectl rollout history deployment/myapp-deployment</code></pre>
<h2>Deployment Strategy</h2>
<ul>
<li><strong>Recreate Strategy:</strong> Destroy all the pods and run the new version of the pods. The application is down during the upgrade.</li>
<li><strong>Rolling Update:</strong> Destroy one pod and create another one (update the pods one by one). This is the default deployment strategy.</li>
</ul>

<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/bfb61f0d-cc04-4564-a2a4-ba9a7a20482a" width="800"/>
</div>
<br/>


<h2>Rollback</h2>
<p>To roll back your updated deployment, run the following command:</p>
<pre><code>kubectl rollout undo deployment/myapp-deployment</code></pre>

<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/445a8678-018e-4742-83ac-acccc5c70a02" width="800"/>
</div>
<br/>


<h2>Force Replace</h2>
<p>To force replace a deployment, run the following command:</p>
<pre><code>kubectl replace --force -f ubuntu-sleeper-3.yaml</code></pre>
<h2>Command and Args in yaml File</h2>

<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/be161289-9908-45af-ac32-26881704f065" width="800"/>
</div>
<br/>


<br/>
<hr/>
<hr/>



<h1>🐳 Environment Variables in Kubernetes</h1>
<p>Environment variables (ENV Vars) in Kubernetes are used to pass configuration data to the containers running in a Pod. They allow you to customize the behavior of applications without changing the code. Here’s a detailed overview:</p>

<h2>Defining Environment Variables</h2>
<p>You can define environment variables in your Pod or container specifications using the <code>env</code> field in a YAML file. Here's an example:</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    env:
    - name: ENV_VAR_NAME
      value: "value"
</code></pre>
<p>In this example:</p>
<ul>
<li>The <code>ENV_VAR_NAME</code> is set to "value" in the container running in the <code>example-pod</code>.</li>
</ul>

<h2>Types of Environment Variables</h2>
<h3>Static Environment Variables</h3>
<p>These are defined with a fixed value directly in the Pod specification, as shown in the example above.</p>

<h3>Dynamic Environment Variables</h3>
<p>These can reference values from Kubernetes resources like ConfigMaps, Secrets, or downward APIs.</p>

<p>Example with ConfigMap:</p>
<pre><code>
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-configmap
data:
  CONFIG_VAR: "config-value"
---
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    env:
    - name: CONFIG_VAR
      valueFrom:
        configMapKeyRef:
          name: example-configmap
          key: CONFIG_VAR
</code></pre>

<h2>Using Secrets for Environment Variables</h2>
<p>To store sensitive information, you can use Kubernetes Secrets:</p>
<pre><code>
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  SECRET_KEY: c2VjcmV0LXZhbHVl # base64 encoded value
---
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    env:
    - name: SECRET_KEY
      valueFrom:
        secretKeyRef:
          name: example-secret
          key: SECRET_KEY
</code></pre>

<h2>Using Downward API</h2>
<p>The Downward API allows you to expose Pod and container fields as environment variables:</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
</code></pre>
<p>In this example, the environment variable <code>POD_NAME</code> will hold the name of the Pod.</p>

<h2>Best Practices</h2>
<ul>
<li><strong>Use ConfigMaps and Secrets:</strong> For better manageability and security, store non-sensitive and sensitive data in ConfigMaps and Secrets.</li>
<li><strong>Keep it Simple:</strong> Avoid complex logic in environment variables; use them for simple configurations.</li>
<li><strong>Base64 Encoding:</strong> Remember to encode sensitive values in Secrets using base64.</li>
</ul>


<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/d01cc848-3dd9-43f1-9cac-443ab3fdd890" width="800"/>
</div>
<br/>



<br/>
<hr/>
<hr/>








<h1>🐳 ConfigMaps</h1>
<p>ConfigMaps in Kubernetes are used to store configuration data in key-value pairs. This allows you to decouple configuration artifacts from image content to keep containerized applications portable. Here's a detailed overview:</p>

<h2>Defining a ConfigMap</h2>
<p>You can define a ConfigMap using a YAML file. Here’s an example:</p>
<pre><code>
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-configmap
data:
  CONFIG_KEY: "config-value"
  ANOTHER_KEY: "another-value"
</code></pre>
<p>In this example, <code>example-configmap</code> contains two keys, <code>CONFIG_KEY</code> and <code>ANOTHER_KEY</code>, with their respective values.</p>

<h2>Using ConfigMaps in a Pod</h2>
<p>To use a ConfigMap in a Pod, you reference the ConfigMap in the Pod’s specification. Here’s an example:</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    env:
    - name: CONFIG_KEY
      valueFrom:
        configMapKeyRef:
          name: example-configmap
          key: CONFIG_KEY
</code></pre>
<p>In this example:</p>
<ul>
<li>The environment variable <code>CONFIG_KEY</code> in the container is populated with the value from the <code>example-configmap</code>.</li>
</ul>

<h2>Using ConfigMaps as Volume Mounts</h2>
<p>ConfigMaps can also be mounted as volumes to provide configuration files to containers. Here’s an example:</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: example-configmap
</code></pre>
<p>In this example:</p>
<ul>
<li>The <code>example-configmap</code> is mounted at <code>/etc/config</code> inside the container.</li>
</ul>

<h2>Best Practices</h2>
<ul>
<li><strong>Separate Configuration:</strong> Store configuration data separately from application code to make applications portable.</li>
<li><strong>Version Control:</strong> Store ConfigMaps in version control systems (e.g., Git) to keep track of changes.</li>
<li><strong>Secure Sensitive Data:</strong> Use Secrets instead of ConfigMaps for sensitive data.</li>
</ul>



<br/>
<hr/>
<hr/>





<h1>🐳 Secrets</h1>
<p>Secrets in Kubernetes are used to store sensitive information such as passwords, OAuth tokens, and SSH keys. Unlike ConfigMaps, which are intended for non-sensitive data, Secrets ensure that sensitive data is stored securely and is only accessible to authorized Pods.</p>

<h2>Defining a Secret</h2>
<p>You can define a Secret using a YAML file. Here’s an example:</p>
<pre><code>
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  USERNAME: dXNlcm5hbWU=  # base64 encoded value
  PASSWORD: cGFzc3dvcmQ=  # base64 encoded value
</code></pre>
<p>In this example, <code>example-secret</code> contains two keys, <code>USERNAME</code> and <code>PASSWORD</code>, with their respective base64 encoded values.</p>

<h2>Using Secrets in a Pod</h2>
<p>To use a Secret in a Pod, you reference the Secret in the Pod’s specification. Here’s an example:</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: example-secret
          key: USERNAME
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: example-secret
          key: PASSWORD
</code></pre>
<p>In this example:</p>
<ul>
<li>The environment variables <code>USERNAME</code> and <code>PASSWORD</code> in the container are populated with the values from the <code>example-secret</code>.</li>
</ul>

<h2>Using Secrets as Volume Mounts</h2>
<p>Secrets can also be mounted as volumes to provide sensitive files to containers. Here’s an example:</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret
  volumes:
  - name: secret-volume
    secret:
      secretName: example-secret
</code></pre>
<p>In this example:</p>
<ul>
<li>The <code>example-secret</code> is mounted at <code>/etc/secret</code> inside the container.</li>
</ul>

<h2>Best Practices</h2>
<ul>
<li><strong>Base64 Encoding:</strong> Ensure sensitive values in Secrets are base64 encoded.</li>
<li><strong>RBAC Policies:</strong> Use Kubernetes Role-Based Access Control (RBAC) to control access to Secrets.</li>
<li><strong>Encryption:</strong> Enable encryption at rest for Secrets in the Kubernetes cluster.</li>
<li><strong>Data Storage:</strong> Secrets store data in an encoded format.</li>
</ul>

<h2>Additional Commands</h2>
<p>To view the details of a secret in YAML format, use the following command:</p>
<pre><code>kubectl get secret app-secret -o yaml</code></pre>
<p>Note that Secrets are not encrypted, only encoded. To encrypt the secrets, use an object called <code>EncryptionConfiguration</code>.</p>
<p>Anyone able to create Pods/Deployments in the same namespace can access the secrets.</p>
</body>
</html>


<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/b979acb3-87c4-488b-b1c3-ad95100ed368" width="800"/>
</div>
<br/>
