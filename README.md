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







