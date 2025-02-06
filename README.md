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



<h1>🐳 Ingress</h1>

<h2>Overview</h2>
<p>Ingress in Kubernetes helps your users access your application using a single externally accessible URL. It allows you to configure routing to different services within your cluster based on the URL path, and it also supports SSL security.</p>

<h2>Ingress as a Load Balancer</h2>
<p>Think of Ingress as a layer seven load balancer built into the Kubernetes cluster. It provides advanced routing capabilities based on HTTP/HTTPS requests.</p>

<h2>Ingress Controller</h2>
<p>An Ingress Controller is like a load balancer that users can access. There are several solutions available, such as NGINX, Istio, HAProxy, and Traefik. These controllers have additional intelligence built into them to monitor the Kubernetes cluster for new definitions or ingress resources.</p>
<p>We deploy the Ingress Controller as a deployment in the Kubernetes cluster. These solutions are not deployed by default in the Kubernetes cluster and must be deployed manually.</p>

<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/fbe0773e-95fc-4ac8-a154-3184f4b71744" width="700"/>
</div>
<br/>

<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/d556b95a-c053-48e4-b29b-15de55b998f0" width="700"/>
</div>
<br/>
<br/>
<div align="center">
    <img src="https://github.com/user-attachments/assets/e98e4314-5ed7-465d-bb33-e6843c550b41" width="700"/>
</div>
<br/>



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



<br/>
<hr/>
<hr/>


<h1>🐳 Multi-Container Pods</h1>
<p>In Kubernetes, it is possible to have pods that contain multiple containers. These multi-container pods share the same lifecycle, meaning they are created together and destroyed together. They also share the same storage and network namespace.</p>
<p>For example, a multi-container pod may consist of a web container and a log container. The containers within the pod can communicate with each other using <code>localhost</code> and the port number on which the service is running.</p>


<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/1733e200-49f6-4eab-ae6c-39ce200b08af" width="800"/>
</div>
<br/>



<br/>
<hr/>
<hr/>






<h1>🐳 Cluster Maintenance in Kubernetes</h1>

<h2>Node Down Scenario</h2>
<p>If a node goes down in the cluster, the following actions occur:</p>
<ul>
    <li>If the node comes back online, the pods on it will also come back online.</li>
    <li>If the node does not come back within 5 minutes, the pods are terminated. If the pods are part of a deployment or replicaset, they will be recreated on other nodes.</li>
    <li>When the node comes back online, it will start without any pods scheduled on it.</li>
</ul>

<h2>Evacuating a Node for Maintenance</h2>
<p>To evacuate a node from the workload for maintenance purposes, use the following command:</p>
<pre><code>kubectl drain node-1</code></pre>
<p>This command gracefully terminates the pods on the node and recreates them on another node. The node is also marked as unschedulable.</p>

<h2>Reactivating a Node</h2>
<p>When the node comes back online, run the following command to make it schedulable again:</p>
<pre><code>kubectl uncordon node-1</code></pre>

<h2>Additional Commands</h2>
<p>To make a node unschedulable, preventing new pods from being scheduled on it, use the following command:</p>
<pre><code>kubectl cordon node-1</code></pre>






<br/>
<hr/>
<hr/>





<h1>🐳 Kubernetes Releases</h1>

<h2>Versioning</h2>
<p>Kubernetes versions follow the pattern <code>V1.11.3</code>, where:</p>
<ul>
  <li><strong>Major Version:</strong> Significant changes and new features (e.g., V1).</li>
  <li><strong>Minor Version:</strong> Introduces new features and functionality, released every few months (e.g., 11).</li>
  <li><strong>Patch Version:</strong> Fixes bugs and issues, released more frequently (e.g., 3).</li>
</ul>

<h2>Version Compatibility</h2>
<p>It is not mandatory to have the same version for all Kubernetes components in the cluster. However, no component should have a version higher than the kube-apiserver.</p>
<p>Only the last three versions of Kubernetes are supported.</p>

<h2>Cluster Upgrade Process</h2>
<p>Upgrading a cluster involves two major steps:</p>
<ul>
  <li>Update the master nodes.</li>
  <li>Update the worker nodes.</li>
</ul>

<h2>Master Node Failure</h2>
<p>If the master node fails, management functions like delete, modify, and deploy new apps are unavailable. However, the workload remains available to users. If a pod in a deployment goes down, it will not be recreated automatically.</p>

<h2>Node Upgrade</h2>
<p>The nodes will not be upgraded until you manually update the kubelet service on each node. Restarting the service will update the versions on the nodes.</p>

<h2>Preparation for Cluster Upgrade</h2>
<p>Before upgrading a cluster, ensure that the repository and key are set correctly. Refer to the Kubernetes documentation for detailed instructions.</p>

<h2>Backup Resources</h2>
<p>To get a backup for all resources, use the following command:</p>
<pre><code>kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml</code></pre>





<br/>
<hr/>
<hr/>



<h1>🐳 Backup and Restore ETCD in Kubernetes</h1>

<h2>Backing Up ETCD</h2>
<p>To take a snapshot of the ETCD cluster, you can use the <code>etcdctl</code> utility. The API uses <code>https://127.0.0.1:2379</code> to access the ETCD database.</p>

<h2>Stopping the API Service</h2>
<p>Before performing certain operations, you may need to stop the API service.</p>

<h2>Restoring the ETCD Cluster</h2>
<p>To restore the ETCD cluster:</p>
<ul>
    <li>Run the <code>etcdctl</code> command as specified in the documentation.</li>
    <li>Configure the ETCD service to use the new directory.</li>
    <li>Restart the daemons and services, then start the kube-apiserver service.</li>
</ul>

<h2>Using <code>etcdctl</code> Options</h2>
<p>Make sure to use the appropriate options when using <code>etcdctl</code>.</p>

<h2>Restoring ETCD</h2>
<p>To restore ETCD:</p>
<ul>
    <li>First, stop the kube-apiserver.</li>
    <li>Move the kube-apiserver manifest from the Static Pod directory to another location. (To find the Static Pod directory, use <code>ps -aux | grep kubelet</code> and check the options.)</li>
    <li>Run the <code>etcdctl</code> command and create a directory as specified in the documentation.</li>
    <li>Update the <code>etcd.yaml</code> file with the new directory.</li>
</ul>

<h2>Kubectl Commands</h2>
<p>To check the kubectl configuration file in the node, use:</p>
<pre><code>kubectl config view</code></pre>
<p>To switch contexts in Kubernetes, use:</p>
<pre><code>kubectl config use-context &lt;name-of-the-context&gt;</code></pre>

<h2>ETCD Configurations</h2>
<ul>
    <li><strong>Stacked ETCD:</strong> ETCD is stacked with Kubernetes components within the cluster.</li>
    <li><strong>External ETCD:</strong> Configure the ETCD database on a separate server.</li>
</ul>

</body>
</html>





<br/>
<hr/>
<hr/>

<h1>🐳 Security in Kubernetes</h1>

<h2>Controlling Access to the API Server</h2>
<p>The first line of defense in Kubernetes security is controlling access to the API server. This involves:</p>
<ul>
  <li><strong>Authentication:</strong> Determining who can access the server.</li>
  <li><strong>Authorization (RBAC):</strong> Determining what actions they can perform.</li>
</ul>

<h2>Network Policies</h2>
<p>By default, all pods can communicate with each other within the cluster. To prevent unauthorized communication, use network policies.</p>

<h2>Service Accounts</h2>
<p>Kubernetes does not manage service accounts directly. Instead, it relies on external sources such as files with user details or certificates.</p>

<h2>Managing User Access</h2>
<p>All user access is managed by the kube-apiserver, whether accessed via the kubectl tool or directly through the API (e.g., <code>curl https://kube-server-ip:6443</code>). The kube-apiserver first authenticates the request and then processes it.</p>
<p>Authentication mechanisms include static password files, certificates, or identity services. For example, static passwords can be set up with the following format: <code>password123,user4,user-id</code>. The file name is then passed as an option to the kube-apiserver using <code>--basic-auth-file=file-name</code>.</p>

<h2>Certificates</h2>
<p>Certificates are a key component of Kubernetes security:</p>
<ul>
  <li><strong>CA (Certificate Authority):</strong> Has a public key (built into browsers) and a private key (used to sign server certificates).</li>
  <li><strong>Server:</strong> Generates a public key (sent to users to encrypt symmetric keys) and a private key (used to decrypt symmetric keys).</li>
  <li><strong>User:</strong> Generates a symmetric key used for ongoing communication.</li>
</ul>
<p>Public and private keys are related; one can encrypt data and only the corresponding key can decrypt it. Note that if data is encrypted with the private key, anyone with the public key can decrypt it.</p>

<h2>Key Extensions</h2>
<ul>
  <li><strong>Public Key:</strong> .cert, .pem</li>
  <li><strong>Private Key:</strong> .key, .key.pem</li>
</ul>





<br/>
<hr/>
<hr/>






<h1>🐳 Certificates in Kubernetes</h1>

<h2>Generating CA Key and Certificate with Self-Sign</h2>
<ol>
    <li>Generate the CA key:
        <pre><code>openssl genrsa -out ca.key 2048</code></pre>
    </li>
    <li>Create a certificate signing request (CSR):
        <pre><code>openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr</code></pre>
    </li>
    <li>Generate the CA certificate:
        <pre><code>openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt</code></pre>
    </li>
</ol>

<h2>Generating Admin User Key and Certificate Signed by the CA</h2>
<ol>
    <li>Generate the admin user key:
        <pre><code>openssl genrsa -out admin.key 2048</code></pre>
    </li>
    <li>Create a CSR for the admin user:
        <pre><code>openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr</code></pre>
    </li>
    <li>Sign the admin user certificate with the CA:
        <pre><code>openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt</code></pre>
    </li>
    <li>Optionally, create a CSR with an organization:
        <pre><code>openssl req -new -key admin.key -subj "/CN=kube-admin/O=system.masters" -out admin.csr</code></pre>
    </li>
</ol>

<h2>Using <code>etcdctl</code> for Authentication</h2>
<p>When using the options <code>--key</code>, <code>--cert</code>, and <code>--cacert</code> with <code>etcdctl</code>, they are used to authenticate requests with ETCD.</p>

<h2>Displaying a Certificate</h2>
<p>To show a specific certificate, use the following command:</p>
<pre><code>openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout</code></pre>

<h2>Troubleshooting <code>kube-apiserver</code> Pod Issues</h2>
<p>If there is a problem with the <code>kube-apiserver</code> pod, check the container log and the ETCD container as the API server gets its data from the ETCD database.</p>

<h2>Kubernetes Certificates API</h2>
<p>Kubernetes has a built-in Certificates API that manages rotating expired certificates and signing new certificates.</p>
<p>Create a <code>CertificateSigningRequest</code> object and import the CSR encoded with base64. You can view the CSR as an object in Kubernetes using:</p>
<pre><code>kubectl get csr</code></pre>
<p>Approve the CSR using:</p>
<pre><code>kubectl certificate approve &lt;name-of-the-csr&gt;</code></pre>

<h2>Useful Hint</h2>
<p>To get the YAML file of any object when you don't know where it is, use the following command:</p>
<pre><code>kubectl get [object] [name-of-the-object] -o yaml</code></pre>





<br/>
<hr/>
<hr/>









<h1>🐳 KubeConfig File</h1>

<h2>Overview</h2>
<p>The KubeConfig file is used to configure access to the Kubernetes API server. It allows you to specify the necessary options for the <code>kubectl</code> utility to interact with the API server without having to provide those options in every command.</p>
<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/1e385559-f56a-4baa-8c16-2cb4f12efc9e" width="800"/>
</div>
<br/>


<h2>Configuration</h2>
<p>When using <code>kubectl</code> to request actions from the Kubernetes API server, you typically need to specify options such as <code>--server</code>, <code>--client-key</code>, <code>--client-certificate</code>, and <code>--certificate-authority</code>. By setting these options in the KubeConfig file, you can use <code>kubectl</code> without specifying them explicitly each time.</p>


<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/579c547f-3ec6-4e85-8924-18ce3e42fd70" width="800"/>
</div>
<br/>


<h2>Default Location</h2>
<p>By default, Kubernetes looks for the KubeConfig file in <code>$HOME/.kube/config</code>. If you create the file in this location, you do not need to specify the path to the file explicitly in the <code>kubectl</code> command.</p>

<h2>Current Context</h2>
<p>To use a specific context as the default, set the <code>current-context</code> in the KubeConfig file:</p>
<pre><code>current-context: dev-user@ggoggole</code></pre>

<h2>Viewing the KubeConfig File</h2>
<p>To view the KubeConfig file, use the following command:</p>
<pre><code>kubectl config view</code></pre>

</body>
</html>



<br/>
<hr/>
<hr/>







<h1>🐳 Roles and RoleBindings</h1>

<h2>Overview</h2>
<p>In Kubernetes, Roles define the permissions to perform specific actions on resources, while RoleBindings link users or groups to the roles created.</p>

<h2>Roles</h2>
<p>Roles specify the objects and the actions that can be performed on them. To view roles in your cluster, use the following command:</p>
<pre><code>kubectl get roles</code></pre>

<h2>RoleBindings</h2>
<p>RoleBindings associate users or groups with the defined roles, effectively granting them the specified permissions. To view role bindings in your cluster, use the following command:</p>
<pre><code>kubectl get rolebindings</code></pre>


<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/e2f9329a-46c4-4d47-b9c8-1ed83ba7d78c" width="800"/>
</div>
<br/>


<h2>Describing a Role</h2>
<p>To get detailed information about a specific role, use the following command:</p>
<pre><code>kubectl describe role developer</code></pre>



<br/>
<hr/>
<hr/>



<h1>🐳 Kubernetes Permissions</h1>

<h2>Checking Permissions for Specific Actions on Objects</h2>
<p>To check whether a specific action on an object is permitted, use the following commands:</p>
<pre><code>
kubectl auth can-i create deployments  # yes
kubectl auth can-i delete nodes  # no
</code></pre>

<h2>Specifying Access for a Specific User</h2>
<p>To specify access permissions for a specific user, use the appropriate <code>kubectl auth</code> commands to verify their permissions.</p>

<h2>Specifying Access to a Specific Object Name</h2>
<p>To specify access to a specific object name, ensure you define roles and role bindings that grant the required permissions to the user or group.</p>


<br/>
<hr/>
<hr/>



<h1>🐳 Cluster Roles and ClusterRoleBindings</h1>

<h2>Overview</h2>
<p>Cluster Roles and ClusterRoleBindings in Kubernetes function similarly to Roles and RoleBindings but apply to cluster-wide permissions. This means they grant access to resources across the entire cluster, not just within a specific namespace.</p>

<h2>Cluster Roles</h2>
<p>Cluster Roles define the permissions to perform specific actions on cluster-scoped resources. These roles can also be created for namespace-scoped resources, granting the user access to all resources in all namespaces.</p>

<h2>ClusterRoleBindings</h2>
<p>ClusterRoleBindings associate users or groups with Cluster Roles, effectively granting them the specified cluster-wide permissions.</p>





<br/>
<hr/>
<hr/>






<h1>🐳 Service Accounts</h1>

<h2>Overview</h2>
<p>A Service Account in Kubernetes is an account used by an application to interact with a Kubernetes cluster. It allows the application to authenticate with the API server and access resources within the cluster.</p>

<h2>Creating a Service Account</h2>
<p>When you create a Service Account using the following command, a token is automatically generated:</p>
<pre><code>kubectl create serviceaccount &lt;name-of-it&gt;</code></pre>



<br/>
<hr/>
<hr/>









<h1>🐳 Images in Kubernetes</h1>
<p>In Kubernetes, images are often stored in private repositories to enhance security and control access. Here’s how you can use images from private repositories in your Kubernetes cluster:</p>

<h2>Step 1: Create a Docker Registry Secret</h2>
<p>Create a Docker registry secret that contains the credentials required to access the private repository. This can be done using the following command:</p>
<pre><code>
kubectl create secret docker-registry myregistrykey --docker-server=&lt;DOCKER_REGISTRY_SERVER&gt; --docker-username=&lt;USERNAME&gt; --docker-password=&lt;PASSWORD&gt; --docker-email=&lt;EMAIL&gt;
</code></pre>
<ul>
    <li>&lt;DOCKER_REGISTRY_SERVER&gt; is the URL of your private Docker registry.</li>
    <li>&lt;USERNAME&gt; is your Docker registry username.</li>
    <li>&lt;PASSWORD&gt; is your Docker registry password.</li>
    <li>&lt;EMAIL&gt; is your Docker registry email.</li>
</ul>

<h2>Step 2: Reference the Secret in Your Pod Specification</h2>
<p>Include the secret in your pod specification under the <code>imagePullSecrets</code> section:</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: &lt;DOCKER_REGISTRY_SERVER&gt;/&lt;IMAGE_NAME&gt;:&lt;TAG&gt;
  imagePullSecrets:
  - name: myregistrykey
</code></pre>
<p>&lt;DOCKER_REGISTRY_SERVER&gt;/&lt;IMAGE_NAME&gt;:&lt;TAG&gt; is the image path and tag in your private repository.</p>
<p>This configuration ensures that Kubernetes will use the provided credentials to pull the image from the private repository.</p>

<h2>Step 3: Verify Access</h2>
<p>Deploy the pod and verify that it is able to pull the image from the private repository without any issues.</p>

</body>
</html>



<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/52600dbe-c996-425b-9a15-e0005b6ff7a5" width="800"/>
</div>
<br/>

<br/>
<hr/>
<hr/>



<h1>🐳 Network Policies</h1>
<p>Network Policies in Kubernetes are used to control the communication between pods. By default, all pods can communicate with each other within the same cluster. Network Policies allow you to specify how groups of pods are allowed to communicate with each other and other network endpoints.</p>

<h2>Defining a Network Policy</h2>
<p>A Network Policy is defined using a YAML file. Here’s an example of a simple Network Policy:</p>
<pre><code>
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: frontend
</code></pre>
<p>In this example:</p>
<ul>
  <li>The Network Policy named <code>example-network-policy</code> applies to the <code>default</code> namespace.</li>
  <li>It selects pods with the label <code>role: db</code>.</li>
  <li>It specifies both ingress and egress rules.</li>
  <li>Only pods with the label <code>role: frontend</code> can communicate with the selected pods.</li>
</ul>

<h2>Applying a Network Policy</h2>
<p>To apply a Network Policy, save the YAML file and use the following command:</p>
<pre><code>kubectl apply -f example-network-policy.yaml</code></pre>

<h2>Types of Network Policies</h2>
<ul>
  <li><strong>Ingress Policy:</strong> Controls incoming traffic to the pods.</li>
  <li><strong>Egress Policy:</strong> Controls outgoing traffic from the pods.</li>
</ul>

<h2>Best Practices</h2>
<ul>
  <li><strong>Least Privilege Principle:</strong> Apply the principle of least privilege by allowing only necessary communication between pods.</li>
  <li><strong>Test Policies:</strong> Test network policies in a staging environment before applying them in production.</li>
  <li><strong>Monitor Traffic:</strong> Use monitoring tools to ensure that the network policies are working as expected and not causing unintended disruptions.</li>
</ul>



<br/>
<hr/>
<hr/>






<h1>🐳 Multi-Node Cluster in Kubernetes</h1>

<h2>API Server</h2>
<p>The API Server can be configured in an active-active status across nodes. A front-line load balancer can be set up to point to the three API servers, distributing the traffic among them.</p>

<h2>Controller Manager and Scheduler</h2>
<p>The Controller Manager and Scheduler are configured in an active-standby status. Through an election process, only one instance is active at a time.</p>

<h2>ETCD</h2>
<ul>
    <li>When reading from the ETCD cluster, you can read from any node in the cluster.</li>
    <li>For writing data, only one node is responsible for write operations. The nodes elect a leader to manage the writes, while the other nodes act as followers.</li>
    <li>The leader ensures that the writes are distributed to other instances in the cluster. A write operation is only considered complete if the leader gets consent from other members in the cluster.</li>
    <li>The nodes use the RAFT protocol for leader election, which employs random timers for initiating requests.</li>
    <li>Quorum is the minimum number of nodes needed in the cluster to function properly or make a successful write. For a cluster of 3 nodes, the quorum is 2, calculated as Q = N/2 + 1.</li>
</ul>

</body>
</html>






<br/>
<hr/>
<hr/>



<h1>🐳 Storage in Kubernetes</h1>
<p>Storage in Kubernetes is designed to provide persistent and temporary storage solutions for applications running in the cluster. Here’s an overview of the different storage options available in Kubernetes:</p>

<h2>Persistent Volumes (PVs)</h2>
<p>Persistent Volumes are storage resources in the cluster that are provisioned by an administrator. They exist independently of pods and provide a way for users to request storage that persists beyond the lifecycle of individual pods.</p>
<pre><code>
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
</code></pre>

<h2>Persistent Volume Claims (PVCs)</h2>
<p>Persistent Volume Claims are requests for storage by a user. They specify the desired capacity and access modes. When a PVC is created, Kubernetes finds a suitable PV to bind it to.</p>
<pre><code>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
</code></pre>

<h2>Dynamic Provisioning</h2>
<p>Dynamic provisioning allows Kubernetes to automatically create a PV for a PVC, eliminating the need for an administrator to manually provision storage. This is achieved using StorageClasses.</p>
<pre><code>
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage-class
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
</code></pre>
<pre><code>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: my-storage-class
</code></pre>

<h2>Configuring Volumes in Pods</h2>
<p>Volumes are used to store data within a pod. Kubernetes supports several types of volumes, such as emptyDir, hostPath, and more.</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: my-volume
  volumes:
  - name: my-volume
    emptyDir: {}
</code></pre>

<h2>Access Modes</h2>
<p>Access Modes define how the PV can be mounted and accessed by pods. Common access modes include:</p>
<ul>
  <li><strong>ReadWriteOnce (RWO):</strong> The volume can be mounted as read-write by a single node.</li>
  <li><strong>ReadOnlyMany (ROX):</strong> The volume can be mounted as read-only by many nodes.</li>
  <li><strong>ReadWriteMany (RWX):</strong> The volume can be mounted as read-write by many nodes.</li>
</ul>

<h2>Storage Best Practices</h2>
<ul>
  <li><strong>Use PVCs:</strong> Always use Persistent Volume Claims to request storage rather than directly specifying PVs.</li>
  <li><strong>Dynamic Provisioning:</strong> Leverage dynamic provisioning to simplify storage management.</li>
  <li><strong>Backup:</strong> Regularly back up your persistent data to avoid data loss.</li>
</ul>

<h2>Additional Concepts</h2>
<ul>
  <li><strong>CRI (Container Runtime Interface):</strong> The standard that defines how Kubernetes communicates with container runtime engines like Docker or CRI.</li>
  <li><strong>CNI (Container Network Interface):</strong> Extends support for different networking solutions.</li>
  <li><strong>CSI (Container Storage Interface):</strong> Enables Kubernetes to work with different storage solutions.</li>
</ul>

<h2>HostPath in Volume Storage Type</h2>
<p>A pod can be mounted to a specific file directory on a node using <code>hostPath</code>. However, it is not recommended in multi-node clusters because if the pod is recreated on another node, the data in the container will not be the same.</p>

<h2>Administrator and Developer Roles</h2>
<p>The administrator creates a set of persistent volumes, and developers create persistent volume claims to use the storage.</p>

<h2>Using PVCs in Pod Definition</h2>
<p>Once you create a PVC, use it in a pod definition file by specifying the PVC claim name under the <code>persistentVolumeClaim</code> section in the volumes section, like this:</p>
<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: my-volume
  volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName: my-pvc
</code></pre>

<h2>Access Mode Importance</h2>
<p>Access mode is very important to ensure the PVC matches the PV. When you delete a PVC used by a pod, the pod will be stuck, and after you delete the pod, it will be deleted.</p>

</body>
</html>

<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/cb518274-2e4f-4f37-9c0b-dbd818dc71e2" width="800"/>
</div>
<br/>

<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/da479447-7f76-4c57-a16d-819eb00247aa" width="800"/>
</div>
<br/>
<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/300a77be-06ad-4870-9a3c-ac08c26a423e" width="800"/>
</div>
<br/>
<br/>
<div align="center">
       <img src="https://github.com/user-attachments/assets/1c3a168f-bfa6-4262-965f-cfe8b0451af3" width="800"/>
</div>
<br/>



<br/>
<hr/>
<hr/>



<h1>🐳 Troubleshooting Issues in Kubernetes</h1>

<h2>Connection Issues</h2>
<ul>
    <li><strong>Connection Refused:</strong> Check the user and password of the database and the deployment.</li>
    <li><strong>Connection Refused:</strong> Verify the ports of the service and the pods.</li>
    <li><strong>Access Denied:</strong> Ensure the database user is correctly configured.</li>
    <li><strong>Error on Gateway:</strong> Check the port of the node in the service deployment file.</li>
</ul>

<h2>File Visibility Issues</h2>
<ul>
    <li>Ensure the file exists.</li>
    <li>Check the permissions on the file.</li>
    <li>Verify the mounts in the pod YAML (host paths).</li>
</ul>

<h2>Node Crash Issues</h2>
<ul>
    <li>Use <code>top</code> and <code>df</code> to check resource usage.</li>
    <li>Check the status of the kubelet.</li>
    <li>Use <code>journalctl -u kubelet</code> to review logs.</li>
    <li>Verify the certificates are not expired and are issued by the correct CA using:<br/>
        <pre><code>openssl x509 /var/lib/kubelet/worker01.crt -text</code></pre>
    </li>
    <li>Check <code>/var/lib/kubelet/config.yaml</code> for configuration issues.</li>
    <li>Ensure the port of the control plane is correct in <code>/etc/kubernetes/kubelet.conf</code> (should be 6443).</li>
</ul>



</body>
</html>


