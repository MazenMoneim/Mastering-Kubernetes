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
  <img src="https://github.com/user-attachments/assets/57d55129-cd1f-49b4-9f7c-31d583a00d56" width="800"/>
</div>

<br/>
<hr/>
<hr/>

<!DOCTYPE html>
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
<!DOCTYPE html>
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
<!DOCTYPE html>
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
<!DOCTYPE html>
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

<!DOCTYPE html>
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


