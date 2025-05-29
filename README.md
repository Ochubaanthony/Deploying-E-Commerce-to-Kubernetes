# Deploying-E-Commerce-to-Kubernetes
Step by steps to deployed Project on Kubernetes


HOW TO CONNECT TO 1 or Many KUBERNETES CLUSTERS from Command line?

On the server where you run 

git clone  https://github.com/iam-veeramalla/ultimate-devops-project-demo
cd ultimate-devops-project-demo





Overview 
n the previous session, we learned how to connect to an EKS cluster and explored kubectl. In this lecture, the focus is on Kubernetes implementation for the project, covering the following key points:
	•	Introduction to Kubernetes concepts like service accounts, deployments, and services.
	•	Explanation of why service accounts are needed in real-world deployments.
	•	Understanding Kubernetes deployments and services to grasp scaling and service discovery.
	•	Project deployment to Kubernetes using predefined YAML files for each microservice (available in the GitHub repo ultimate-devops-project-demo).
	•	Overview of the deployment and service files for various microservices.
	•	Learning how to expose the application to the external world using:
	•	LoadBalancer service
	•	Ingress and Ingress Controller
	•	Detailed steps provided for setting up ingress and accessing the project externally.
	•	Demonstration of using a custom domain with AWS Route 53, integrating it with Ingress for real-world scenarios.
	•	End-to-end implementation ensures a comprehensive understanding of Kubernetes in a real project.
	•	Upcoming in the CI/CD section: Deploying the app to Kubernetes using Argo CD (part of CD, not this section).
	•	Beginners are encouraged to watch the Kubernetes Zero to Hero and Docker series for foundational understanding.

Conclusion: This lecture sets up a full Kubernetes deployment pipeline with real-world practices and prepares you for interviews.



Class 2
Kubernetes server Account and its importance in realtime


When deploying demo applications as pods in Kubernetes, many skip creating a service account, and Kubernetes still allows the pod to run. This is because Kubernetes automatically assigns a default service account (default) to every pod if no specific one is provided.

Key Concepts:
	•	User Account vs. Service Account:
	•	User Account: For humans (e.g., DevOps, developers) to access the cluster using kubectl and kubeconfig.
	•	Service Account: For services (like pods) to interact with Kubernetes resources.
	•	Why Service Accounts Matter:
	•	Just like users need permissions, pods also need permissions to interact with the Kubernetes API.
	•	The default service account provides minimal permissions (enough to run the pod).
	•	For advanced actions (e.g., accessing ConfigMaps, calling the Kubernetes API), the pod must be assigned a custom service account with specific permissions.

Permissions and Role Binding:
	•	To grant additional permissions:
	•	Create a Role or ClusterRole.
	•	Bind it to the service account using RoleBinding or ClusterRoleBinding.
	•	This is similar to AWS IAM:
	•	Service accounts are like IAM roles.
	•	Permissions are defined in roles (like IAM policies).
	•	Roles are linked to service accounts through bindings.

Important Notes:
	•	In this project, custom roles or bindings are not required, as the microservices are independent and do not need to access the Kubernetes API.
	•	For more advanced Kubernetes development (e.g., controllers, webhooks), using a custom service account with proper permissions is essential.

⸻

Conclusion:
Even though the default service account is sufficient for simple applications, in real-world environments and production deployments, it’s important to create and assign custom service accounts with appropriate permissions to ensure secure and functional Kubernetes operations.

Class 3
Kubernetes Deployment -[scaling and Healing]

Here’s a clear and concise summary of the lecture on Kubernetes Deployments:

⸻

🔁 Overview

Before deploying the project on EKS, it’s essential to understand two core Kubernetes resources:
	1.	Deployment – handles scaling and auto-healing.
	2.	Service – manages service discovery (covered in the next lecture).

⸻

⚠️ Problem with Plain Containers
	•	Containers (via Docker/Podman) are ephemeral — they don’t restart automatically if they crash.
	•	IP addresses change on restarts, making networking and service communication unreliable.
	•	During high traffic (e.g., a sales event), a single container can’t handle the load; we need multiple copies and load balancing.

⸻

🚀 Solution: Kubernetes Deployment

The Deployment resource solves this by enabling:
	•	Scaling: Set the number of pod replicas (e.g., 3, 4, 10, 100) to handle more traffic.
	•	Auto-healing: If a pod crashes, Kubernetes automatically replaces it to maintain the desired replica count.

🔧 How it works:
	•	You create a Deployment YAML with a specified replicas count.
	•	Kubernetes creates a ReplicaSet (a controller) behind the scenes.
	•	The ReplicaSet maintains the desired number of Pods.
	•	If a pod fails, it immediately creates a new one, ensuring uptime.

⸻

💡 Key Takeaways:
	•	Pod: The basic unit in Kubernetes (holds one or more containers).
	•	ReplicaSet: Ensures the desired number of pod replicas are running.
	•	Deployment: Manages the ReplicaSet and provides scaling + healing features.
	•	Why it’s better than Docker alone: You get resilience and scalability out-of-the-box.

⸻

🔜 Next Topic:

In the upcoming lecture, you’ll learn about the Service resource, which enables service discovery—essential for microservices to find and communicate with each other.

Let me know if you’d like a YAML template or diagram for Deployments!



Class 4
Kubernetes service [service discovery]

Here’s a clear and concise summary of the lecture on Kubernetes Services:

⸻

🔁 Recap from Last Lecture
	•	Deployment resource handles scaling and auto-healing.

⸻

🌐 Current Topic: Kubernetes Service
	•	Problem Solved: Service Discovery
	•	In Docker-only setups, containers get new IPs when restarted.
	•	If front-end relies on a fixed back-end IP, communication breaks when back-end IP changes.
	•	Manual updates and restarts are error-prone and inefficient.

⸻

🚀 How Kubernetes Solves This
	•	Kubernetes introduces a Service resource to solve the service discovery issue.

🔧 How it works:
	1.	Deployments create ReplicaSets, which in turn create Pods (front-end and back-end).
	2.	Each Pod gets a label (like a tag/ID).
	3.	Service acts as a stable proxy or load balancer between Pods.

🔑 The service does not use IP addresses to locate Pods. It uses labels.

	•	When a back-end Pod crashes and is replaced (with a new IP), the label remains the same.
	•	The Service watches the label and redirects traffic to the new Pod automatically.

⸻

💡 Key Takeaways:
	•	Front-end talks to the Service, not directly to the back-end Pod.
	•	The front-end’s environment variable points to the Service name, not an IP.
	•	The Service then finds and routes traffic to the correct Pod using label selectors.
	•	This ensures reliable communication even if Pods restart or scale.

⸻

🔗 Comparison:

Feature	Docker-Only Setup	Kubernetes with Services
Pod/Container IP	Changes on restart	Irrelevant (label-based routing)
Communication	IP-dependent, fragile	Label-based, stable
Recovery from Crash	Manual fix/update needed	Handled automatically by Service


⸻

🧠 Summary:
	•	Deployment solves: Scaling & Auto-Healing
	•	Service solves: Service Discovery & Reliable Communication
Together, they make Kubernetes far more robust than standalone container tools.

⸻




Class 5
Deploying the project on kubernetes -Part1

Here’s a clear and structured summary of the lecture on Deploying Microservices in Kubernetes Using Deployment, Service, and ServiceAccount:

⸻

🎓 Recap of Previous Lectures
	•	Learned about:
	•	Deployment: Manages replicas and ensures high availability.
	•	Service: Solves the problem of service discovery.
	•	ServiceAccount: Provides access control and identity to pods.

⸻

🚀 Current Focus: Deploying a Real Project on EKS
	•	The project consists of ~20 microservices.
	•	Creating YAML files for each manually would take hours.
	•	Good news: All manifests are already created and available in the GitHub repo ultimate-devops-project-demo.

⸻

📁 Manifest Structure in the GitHub Repo
	1.	Folder per Microservice:
	•	Example: add, checkout, etc.
	•	Each contains its own:
	•	deployment.yaml
	•	service.yaml
	2.	Single Combined File:
	•	complete-deploy.yaml (~1907 lines).
	•	Includes manifests for all microservices.
	•	Separated by --- (YAML multi-document format).

You can choose:
	•	kubectl apply -f complete-deploy.yaml to deploy everything at once.
	•	Or apply microservice by microservice for better understanding.

⸻

📄 Structure of a Deployment YAML
	•	Common Fields Across All Resources:
	•	apiVersion
	•	kind
	•	metadata: includes name, labels, and annotations
	•	Spec Section (specific to resource type):
	•	For Deployment:
	•	replicas: number of pod replicas (managed by ReplicaSet)
	•	template:
	•	metadata: includes pod-level labels (used by Services)
	•	spec:
	•	containers: container settings (image, ports, env vars, etc.)

🧠 Tip: The template inside spec defines the Pod template—like Docker Compose in structure.
Labels here are critical for service discovery.

⸻

🏷️ Labels for Service Discovery
	•	Example label for the pod:

app.kubernetes.io/name: add-microservice


	•	This is how the Service resource locates the correct pod, even if its IP changes.

⸻

✅ Key Takeaways

Concept	Purpose
Deployment	Ensures scaling and availability via ReplicaSet
Service	Provides stable access to pods via labels
ServiceAccount	Defines identity/access for pods
Metadata & Labels	Used for both identification and routing


⸻

🧪 What’s Next
	•	In the following steps/lectures, you’ll:
	•	Examine one microservice’s YAML in detail.
	•	Understand how to write deployment.yaml and service.yaml.
	•	Deploy on EKS using either full or microservice-by-microservice approach.

🎯 Objective

Deepen your understanding of how to define a complete Kubernetes Deployment manifest — especially focusing on pod-level configuration, including containers, service accounts, ports, environment variables, and more.

⸻

🧱 Core Structure of a Kubernetes Deployment Manifest

✅ 1. Top-Level Fields (Standard for all K8s resources):

apiVersion: apps/v1
kind: Deployment
metadata:
  name: <deployment-name>
  labels:
    <key>: <value>

	•	apiVersion, kind, metadata — common to all K8s resources.
	•	Labels here are NOT used for service discovery — only for organization & identification.

⸻

✅ 2. Spec Section (Deployment-Level)

spec:
  replicas: <number>
  template:
    metadata:
      labels:
        <key>: <value>
    spec:
      serviceAccountName: <account-name>
      containers:
      - name: <container-name>
        image: <image>
        ports:
        - containerPort: <port>
        env:
        - name: <env-name>
          value: <env-value>

	•	replicas: Number of pod replicas.
	•	template: Defines pod configuration.
	•	metadata.labels: Used by Service to find the pod (key for service discovery).
	•	spec (inside template): Core pod configuration:
	•	serviceAccountName: Custom SA or defaults to default if unspecified.
	•	containers: A list (start with -) of container specs (name, image, ports, env vars, etc.).

⸻

🧠 Key Learnings & Best Practices

Concept	Summary
ServiceAccount	Must be explicitly defined to override default.
Container Definition	Must include name, image, port, and optional env vars, volumes, and resource limits.
Image Sources	Though only a few microservices were containerized in the lectures, the GitHub repo already includes correct image names for all services (from Docker Hub or GHCR).
Environment Variables	Often look intimidating — but are usually provided by developers (e.g., for service URLs, credentials, etc.).
Similarity to Docker Compose	Structure is almost identical (name, image, ports, env, volumes). Writing manifests becomes easier with experience.


⸻

🛑 Common Misconceptions
	•	“It’s too long!”: A 60-line YAML doesn’t mean you have to manually write 60 lines — you copy, tweak, and reuse most parts.
	•	“I must know all env vars!”: No — you’re responsible for the structure. Dev teams provide environment-specific details.

⸻

📌 Pro Tips
	•	Understand the skeleton:
	•	apiVersion, kind, metadata, spec
	•	Within spec: replicas, template, and inside template.spec: pod-level config
	•	Focus on the fields that vary:
	•	Image name
	•	Ports
	•	Env vars
	•	Volumes
	•	Refer to existing YAMLs and the Kubernetes documentation for field definitions.

⸻

🧪 What Should You Do Now?
	•	Practice by reading deployment YAMLs for different microservices (e.g., currency, add, etc.).
	•	Recognize patterns — the structure is consistent.
	•	Experiment by editing values (e.g., change replicas, image versions) and apply them on a test cluster.

⸻



Class 6
Deploying the project on Kubernetes -Part2


🎯 Objective

Understand the structure of a Kubernetes Service resource and how it connects to pods defined in a Deployment.

⸻

🧱 Core Structure of a Kubernetes Service Manifest

✅ 1. Top-Level Fields

apiVersion: v1
kind: Service
metadata:
  name: <service-name>
  labels:
    <key>: <value>

	•	apiVersion: Always v1 for services.
	•	kind: Always Service.
	•	metadata: Includes name and optional labels for identification and troubleshooting.

⸻

✅ 2. Spec Section (Service-Level Configuration)

spec:
  type: ClusterIP | NodePort | LoadBalancer
  ports:
  - port: <service-port>
    targetPort: <container-port>
  selector:
    <label-key>: <label-value>

Field	Meaning
type	How the service is exposed (internal or external):ClusterIP (default, internal use), NodePort, LoadBalancer
port	The port on which the service will be available (can be arbitrary)
targetPort	The container’s port in the pod to forward requests to (must match pod’s config)
selector	The label the service uses to find the target pod(s)


⸻

🔄 How Services Link to Deployments
	•	Service uses selector to find pods.
	•	The label in the selector must match a label defined in the pod template inside the deployment.
	•	The target port must match the container port defined in the deployment.

📌 Example Connection:
	•	Deployment YAML:

spec:
  template:
    metadata:
      labels:
        app: currency-service
    spec:
      containers:
      - name: currency
        image: currency:v1
        ports:
        - containerPort: 8080


	•	Service YAML:

spec:
  type: ClusterIP
  selector:
    app: currency-service
  ports:
  - port: 8080
    targetPort: 8080



⸻

🧠 Key Takeaways

Concept	Summary
Selector	The critical bridge — helps the service know which pod(s) to forward traffic to.
Port vs TargetPort	port is the exposed port by the service; targetPort is the port inside the pod (must match container definition).
Service Name as FQDN	Other microservices talk to it via: service-name.namespace.svc.cluster.local:<port>
Match Labels Carefully	If selector labels don’t match pod labels, the service won’t work.
Documentation Is Key	Kubernetes docs are your friend — nobody memorizes everything at first. You get better with practice.


⸻

📚 Assignment

Explore the Kubernetes manifests in your project:
	1.	For each microservice, open both:
	•	deployment.yaml
	•	service.yaml
	2.	Identify:
	•	The labels on the deployment’s pod template
	•	The selector in the service
	•	Match the targetPort with the containerPort
	3.	Understand how each service is tied to its respective pod.

⸻

🧪 Next Steps (Teaser for Next Lecture)

You’ll soon:
	•	Deploy your microservices using kubectl apply -f <manifest>.
	•	Observe whether resources are created successfully.
	•	Learn basic debugging if things go wrong.

⸻

Class 7
Deploying the project on Kubernetes -Part 3


🎯 Objective

Understand why deployed services aren’t accessible externally yet, and how Kubernetes service types and networking affect that accessibility.

⸻

🔗 Internal Service Communication Recap
	•	In Kubernetes, services communicate using internal DNS-based service discovery.
	•	Example interview Q&A:
Q: How does one microservice connect to another?
A: For example, the shipping service connects to the code service using environment variables that store the service name of the code service. Kubernetes DNS resolves this name internally.
	•	Service Discovery via Environment Variables is a common pattern (used instead of or alongside ConfigMaps).

⸻

🚫 Why You Can’t Access the Project Yet

🧪 Attempt:
	•	You run:

kubectl get svc | grep frontend-proxy


	•	You find an IP like 172.x.x.x and port 8080.
	•	You try to open it in your browser using:

http://172.x.x.x:8080



❌ Result:
	•	You can’t access it. Why?

⸻

🧱 Understanding the Network Setup
	•	Your EKS cluster is inside a VPC created using Terraform.
	•	It contains both private and public subnets.
	•	Your Kubernetes services (e.g., frontend-proxy) are currently exposed using the ClusterIP type.
	•	ClusterIP is only accessible inside the cluster.
	•	You’re outside the VPC, so you can’t reach these services directly from your browser.

⸻

⚙️ What’s Missing?

To access your services externally, you need to expose them through a service type that allows external access.

⸻

📌 Preview of the Next Step

You will:
	•	Learn how to change the service type from ClusterIP to:
	•	NodePort (basic external access through worker node IP and port)
	•	or LoadBalancer (provision a cloud load balancer with a public IP)

This allows the frontend (or frontend-proxy) to be accessible from the public internet.

⸻

🧠 Key Takeaways

Concept	Summary
Internal Communication	Done via service names in env variables (Kubernetes service discovery)
ClusterIP Limitation	Only works within the cluster (no external access)
VPC Isolation	Your services are in a private network – browser access won’t work unless exposed
Next Step	Learn about and apply the correct service type to expose frontend



Class 8
Kubernetes Service Types

Here’s a clear and organized summary of the lecture focused on Kubernetes service types and external access, especially using NodePort:

⸻

🎯 Objective

Understand why the frontend is not accessible externally after deployment, and how to expose services using different Kubernetes service types — especially the NodePort type.

⸻

🔐 Problem Recap
	•	You deployed the project and tried to access the frontend via service IP + port.
	•	It didn’t work because:
	•	The service is using ClusterIP (default service type).
	•	ClusterIP is only accessible inside the Kubernetes cluster.
	•	The frontend cannot be accessed from outside the cluster — not even by resources within the same VPC (like EC2 or Lambda).

⸻

📦 Kubernetes Service Types Overview

Type	Access Scope	Use Case
ClusterIP	Internal cluster only	Pod-to-pod or service-to-service (e.g., databases)
NodePort	External access via node IP	Allow access within VPC or from external systems via EC2 public IP
LoadBalancer	Public internet via cloud load balancer	Best for production-ready external services


⸻

🧠 ClusterIP - Default Behavior
	•	Creates a service that is only reachable from within the Kubernetes cluster.
	•	Useful for internal services with sensitive data.
	•	Even other AWS resources inside the same VPC (like EC2 or Lambda) cannot access services with ClusterIP directly.

⸻

🌐 NodePort - Opening Access via Node
	•	By changing service type to NodePort, Kubernetes:
	•	Allocates a port (e.g., 30000–32767) on each node’s IP address.
	•	Updates internal iptables via kube-proxy to route traffic from that node port to the service/pod.
	•	Now, resources inside the VPC (like EC2) can access the service using:

<Node IP>:<NodePort>


	•	Examples:
	•	Frontend service becomes available at NodeIP:33000
	•	Product catalog becomes available at NodeIP:34000
	•	Cart service at NodeIP:36000
	•	This allows limited external access, without using a load balancer.

⸻

🧱 Why This Works
	•	Every node in a Kubernetes cluster is essentially an EC2 instance.
	•	When a service is exposed using NodePort, it can be accessed using the public or private IP of the node — depending on how your networking is configured.

⸻

🔍 Important Notes
	•	NodePort opens ports on all nodes in the cluster, regardless of where the pod actually runs.
	•	Good for development or internal access, but not ideal for production due to:
	•	Limited range of ports
	•	No load balancing across nodes
	•	Potential security risks

⸻

📌 Next Step

You will learn how to use the LoadBalancer service type, which is more scalable and secure for production, and allows access via a cloud-managed public IP or DNS.

⸻



🌍 Objective

Enable external access (from the public internet) to a service running inside a Kubernetes cluster by using the LoadBalancer service type.

⸻

🧱 Context Recap
	•	So far, you’ve seen:
	•	ClusterIP → internal-only access (within cluster)
	•	NodePort → access via Node IP (within VPC or organization)
	•	But what if you want public access, i.e., anyone on the internet should reach your service?

⸻

⚙️ LoadBalancer Service Type
	•	Set type: LoadBalancer in your Service.yaml.
	•	Kubernetes understands this as: “I want external access.”

✅ What Happens Internally:
	1.	API Server detects the change to LoadBalancer.
	2.	It calls a component called CCM (Cloud Controller Manager).
	3.	CCM communicates with your cloud provider (e.g., AWS, Azure, GCP).
	4.	The cloud provider creates a Load Balancer (e.g., an AWS ELB).
	5.	An external IP or FQDN is assigned and attached to your Kubernetes service.
	6.	This IP/DNS can be accessed from anywhere on the internet.

⸻

☁️ Cloud Provider Dependency
	•	Works only on cloud platforms that support CCM.
	•	Examples:
	•	✅ AWS (EKS), Azure (AKS), GCP (GKE)
	•	❌ Minikube, Kind, or K3s (local clusters) → Do not support CCM
	•	Changing to LoadBalancer in these cases does nothing

⸻

🛠️ Summary of Service Types

Service Type	Access Level	Use Case Example
ClusterIP	Only inside Kubernetes cluster	Internal services (e.g., databases, APIs)
NodePort	Inside the VPC or organization network	Dev/test services, internal apps
LoadBalancer	Public internet access via cloud LB	Production apps exposed to the world


⸻

🔜 Next Step



Class 9
Accessing the project using Kubernetes LB Service Type

Terminal
ubuntu@ip-172-31-38-42:~/ultimate-devops-project-demo/kubernetes$ kubectl get svc |grep frontendproxy

kubectl edit svc opentelemetry-demo-frontendproxy              			Typy = “change clusterIP”>  to>  LoadBalancer
Save with this
esc  :wq!   Enter.                “ wait for 5 mins”

BUT IF RUN THIS CODE IMMEDATILY YOU SAVE THE CODE
kubectl get svc  opentelemetry-demo-frontendproxy

NOTE TO CONFIRM YOU HAVE LOAD BALANCER ON EC2 INSTANCE
Go to aws console
Ec2 instance
Load balancer of the same region us-west-2 					“ you see it”
Copy DNS PASTER ON BROSWER
 a242bc24031814f41825f3c38b22c152-993543253.us-west-2.elb.amazonaws.com:8080



Class 10
Disadvantage of LoadBalancer service type


🚫 Drawbacks of Using LoadBalancer Service Type in Kubernetes

1. ❌ Lack of Declarative Configuration
	•	LoadBalancer is created automatically via CCM (Cloud Controller Manager).
	•	You cannot configure things like:
	•	TLS certificates (HTTPS)
	•	Load balancing algorithm
	•	Health checks, security settings, etc.
	•	Any modifications (e.g., adding HTTPS via ACM) must be done manually in the cloud provider’s console, not through code (YAML).
	•	This breaks Infrastructure-as-Code (IaC) best practices — changes are not trackable in Git or version-controlled systems.

⸻

2. 💸 Not Cost-Effective
	•	If you expose 10 services using LoadBalancer, Kubernetes will create 10 individual cloud load balancers (e.g., ALBs).
	•	Load balancers on AWS are expensive.
	•	There’s no built-in support for sharing one load balancer across multiple services with different routes or paths (e.g., /api, /auth, /frontend).

⸻

3. 🔒 Vendor Lock-in / Lack of Flexibility
	•	You’re tied to the cloud provider’s default load balancer, e.g.:
	•	AWS ➝ ALB
	•	Azure ➝ Azure Load Balancer
	•	You cannot use alternative solutions like:
	•	NGINX Ingress Controller
	•	Traefik
	•	F5
	•	You’re limited to what CCM supports.

⸻

4. 🚫 Doesn’t Work Without CCM
	•	In local clusters (e.g., Minikube, Kind, K3s), where Cloud Controller Manager is absent, setting type: LoadBalancer does nothing.
	•	No external IP is created, and the service remains inaccessible from outside.

⸻

✅ Why Ingress Is Better

Feature	LoadBalancer	Ingress
Declarative Configuration	❌ Manual edits required	✅ Fully declarative (YAML)
Cost Efficient	❌ Multiple LBs needed	✅ One LB for many services
TLS/HTTPS Support	❌ Manual via cloud UI	✅ Via Ingress annotations
Flexibility	❌ Only cloud LB allowed	✅ Use NGINX, Traefik, etc.
Works Locally	❌ No CCM, no effect	✅ Works with local setups


⸻

🧭 Conclusion

Although LoadBalancer type allows public access, it is:
	•	Hard to maintain
	•	Expensive at scale
	•	Not declarative
	•	Tied to your cloud provider

Ingress is the recommended, scalable, and declarative solution for managing external access to services in Kubernetes.





✅ Why Ingress is Better than LoadBalancer

Pros of Ingress & Ingress Controller:
	1.	Highly Flexible:
	•	You can choose your own load balancer (e.g., NGINX, Traefik, F5, Envoy).
	•	Just deploy the corresponding Ingress Controller.
	2.	Cloud-Independent:
	•	Ingress does not require Cloud Controller Manager (CCM).
	•	Works on Minikube, K3s, Kind, and other non-cloud setups.
	•	Can expose services using reverse proxy mechanisms even without a cloud provider.
	3.	Declarative and Scalable:
	•	All configurations are done using YAML files.
	•	Easy to maintain, version control, and audit.
	•	Can handle multiple routes and services under one external IP.

⸻

❌ Cons of LoadBalancer Service Type (Recap):
	•	Not declarative (requires manual changes in cloud console).
	•	Expensive if you expose many services (1 load balancer per service).
	•	Tied to the cloud provider’s default LB (e.g., only ALB on AWS).
	•	Doesn’t work in environments without CCM.

⸻

✅ Only Advantage of LoadBalancer Service Type:
	•	Simplicity:
	•	Extremely easy to configure.
	•	No need to deploy additional components like an Ingress Controller.
	•	Just set type: LoadBalancer in the service YAML and it works (in supported environments).

🔸 Ideal for quick testing, prototypes, or small-scale deployments.

⸻

🧠 Takeaway for Interviews:

If asked whether LoadBalancer service type has any benefits:

“Yes, it’s very easy to set up and doesn’t require additional resources like an Ingress Controller. However, it lacks flexibility, is not declarative, and can be expensive at scale. That’s why Ingress is the preferred approach in production.”



Class 11
Understanding Ingress and Ingress Controllers Part 1
Validate this run

nslookup amazon.com
 https://54.239.28.85						Ubable to view the url do to security

nslookup a242bc24031814f41825f3c38b22c152-993543253.us-west-2.elb.amazonaws.com

COPY THE IP Address and add port 8080
34.214.57.106:8080						able to view the URL which is RED FLAG




Here’s a summary of this lecture on Ingress and Ingress Controllers in Kubernetes:

⸻

🔁 Recap:
	•	Previously, we accessed a deployed project via a LoadBalancer service type.
	•	We discussed the drawbacks of using that approach.

⸻

📥 What is Ingress?
	•	In networking, Ingress refers to incoming traffic.
	•	In Kubernetes, Ingress is a Kubernetes resource (like Deployment or Service).
	•	It allows you to define routing rules for incoming traffic to your services or applications.

⸻

📦 Why Ingress Is Useful:
	•	When a frontend service is deployed in an EKS cluster, by default it’s ClusterIP (accessible only within the cluster).
	•	To make it accessible externally, we used a LoadBalancer, which:
	•	Routes traffic from the internet to the service.
	•	Gets a DNS name from AWS (via Cloud Controller Manager).
	•	Connects to the frontend service through private subnet and exposes it via public subnet.

⸻

🚫 Problem with This Approach:
	•	The AWS LoadBalancer gives a random DNS name (e.g., ab123456.elb.amazonaws.com).
	•	If someone does a DNS lookup (nslookup) on this domain, they get the IP address.
	•	Typing the IP address directly in the browser still allows access to the app (http://<IP>:8080).
	•	Security Issue: This is not acceptable in real-world companies.
	•	Companies want users to access their services only via their domain name (e.g., amazon.com), not via IP.

⸻

✅ Solution: Use Ingress
	•	To define strict routing rules (e.g., allow access only via a specific domain), use Ingress.
	•	Ingress lets you:
	•	Specify hostnames (e.g., shop.mycompany.com)
	•	Route traffic based on path or host
	•	Improve security and control over traffic.

⸻

🧠 Takeaway:

“Ingress is the right way to control and route incoming traffic in Kubernetes. It allows organizations to enforce domain-based access and avoid exposing their applications via raw IP addresses, which is a security risk.”



🔄 Recap:
	•	Ingress is required when you want to define routing rules for incoming traffic.
	•	Kubernetes’ Service resources (like LoadBalancer) cannot define detailed routing rules like host/path-based access.

⸻

⚙️ How It Works:

🔹 Service of Type LoadBalancer:
	•	You define the service with type LoadBalancer.
	•	CCM (Cloud Controller Manager) creates a load balancer.
	•	Simple to set up, but lacks routing control (e.g., cannot restrict access by domain or path).

🔹 Ingress + Ingress Controller:
	•	You define an Ingress resource, which:
	•	Specifies which service it targets (e.g., frontend).
	•	Defines host-based and/or path-based routing (e.g., only allow amazon.com/xyz).
	•	The Ingress Controller (a running component in the cluster):
	•	Reads the Ingress YAML file.
	•	Creates and configures a load balancer based on the routing rules.
	•	Acts like CCM, but offers more flexibility in how traffic is routed.

⸻

📘 Example:

A sample Ingress resource may define:

spec:
  rules:
    - host: amazon.com
      http:
        paths:
          - path: /xyz
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 8080

This means:
	•	Requests to amazon.com/xyz are routed to the frontend service on port 8080.

⸻

💡 Key Points:
	•	Ingress is needed only when you want to expose a service externally with specific rules.
	•	You don’t need Ingress for internal services like backend or email services.
	•	In a real-world setup, typically only the frontend (or API gateway) gets an Ingress resource.
	•	Ingress resource does not create a load balancer by itself — the Ingress Controller reads it and provisions one.

⸻

📌 Interview Tip:

If asked “When do you create an Ingress?”:

“You create an Ingress only when you need to expose a service externally and want to define domain- or path-based routing rules. Typically, only the frontend service gets an Ingress resource.”

CLASS 12
Understanding Ingress and Ingress Controllers Part 2


📌 What Is an Ingress Controller?
	•	An Ingress Controller is a Kubernetes controller that:
	•	Watches for Ingress resources in the cluster.
	•	Reads routing rules defined in them.
	•	Creates and configures a load balancer accordingly.

⸻

❗ Important Note:

Ingress Controller is not included by default in Kubernetes.

⸻

🧩 Why Not Included by Default?
	•	Kubernetes is not opinionated about which load balancer to use.
	•	It gives you the freedom to choose based on your needs.

⸻

⚙️ Examples of Ingress Controllers:

Load Balancer You Want	Ingress Controller to Use
NGINX	NGINX Ingress Controller
AWS ALB	AWS ALB Ingress Controller
Traefik	Traefik Ingress Controller
Kong	Kong Ingress Controller
F5	F5 Ingress Controller

	•	These controllers are typically open-source Go programs, maintained by the respective vendors.

⸻

🔁 How It Works:
	1.	You deploy a specific Ingress Controller (e.g. ALB Controller).
	2.	You write an Ingress Resource for your service (e.g. frontend).
	3.	The Ingress Controller reads that Ingress Resource YAML.
	4.	It provisions a Load Balancer (e.g. AWS ALB) with the specified rules (like domain or path restrictions).
	5.	The Load Balancer only allows traffic based on those rules.

⸻

🛠 Project Plan (as per the course):
	•	Step 1: Deploy the AWS ALB Ingress Controller.
	•	Step 2: Create the Ingress Resource for the frontend service.
	•	Step 3: Access the service via the domain name defined in the Ingress resource (not via random IP/DNS).

⸻

💬 Key Takeaway:

Deploying an Ingress resource alone does nothing.
You must have an Ingress Controller running to interpret it and create the corresponding Load Balancer.


Class 13

REPO
https://github.com/iam-veeramalla/ultimate-devops-project-aws/blob/main/14-kubernetes-ingress-controller.md


📌 Goal:

Install the ALB (Application Load Balancer) Ingress Controller in an EKS cluster to route external traffic based on Ingress rules.

⸻

🧠 Key Concept: IAM OIDC Provider
	•	Problem:
The ALB Controller (which runs as a Pod inside your EKS cluster) needs permission to create AWS resources (like Load Balancers), which are outside the Kubernetes cluster.
	•	Solution:
Use IAM Roles for Service Accounts (IRSA):
	•	Create an IAM Role with necessary permissions.
	•	Bind this IAM Role to the Kubernetes Service Account used by the ALB Controller.
	•	Use an OIDC (OpenID Connect) Provider to securely associate them.
	•	This setup allows the ALB Controller pod to act with AWS IAM permissions without hardcoding credentials.

⸻

🪜 Installation Steps (High-Level):

1. 📂 Check Documentation
	•	Refer to the official project docs:
kubernetes-ingress-controller.md
(File name may vary as it gets updated frequently)

2. 📛 Export Cluster Name

export CLUSTER_NAME=my-eks-cluster

3. 🔍 Fetch OIDC ID for the Cluster

OIDC_ID=$(aws eks describe-cluster \
  --name $CLUSTER_NAME \
  --query "cluster.identity.oidc.issuer" \
  --output text | cut -d '/' -f 3)

4. 🔗 Associate IAM OIDC Provider to the Cluster

eksctl utils associate-iam-oidc-provider \
  --cluster $CLUSTER_NAME \
  --approve

This step allows EKS service accounts to assume IAM roles via OIDC.

⸻

💬 Interview Insight:

Q: How can a Kubernetes Pod in EKS create AWS resources like Load Balancers?
A: By using IAM Roles for Service Accounts (IRSA) connected via IAM OIDC provider, which lets the pod securely assume an IAM Role with appropriate policies.

⸻

✅ Next Steps in the Course:
	1.	Install the ALB Ingress Controller.
	2.	Create Ingress Resource for the frontend service.
	3.	Access the frontend using the specified domain name.

Objective:

Install the ALB Controller by:
	1.	Creating an IAM policy with ALB-related permissions.
	2.	Creating an IAM role and attaching it to the Kubernetes service account used by the ALB Controller.

⸻

🪜 Step-by-Step Summary:

1. 📄 Create IAM Policy
	•	The ALB Controller requires specific permissions to manage Elastic Load Balancers.
	•	These permissions are not manually written — they are provided by AWS.

🧰 Download Policy JSON:

curl -o iam-policy.json \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

	•	This file contains all the required IAM permissions for ALB operations.

📤 Create the IAM Policy in AWS:

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam-policy.json

	•	This uploads the policy and makes it available for attachment to IAM roles.

⸻

2. 🧑‍💼 Create IAM Role and Attach to Kubernetes Service Account
	•	The ALB Controller pod runs inside Kubernetes using a service account.
	•	To let the pod assume AWS permissions, we bind the service account to the IAM role using OIDC.

Steps for creating the role and binding it to the service account are provided in the documentation (continuation of what comes next).

⸻

💬 Interview Tip:

Q: How does the ALB Ingress Controller pod gain permission to create AWS Load Balancers?

A:
	1.	AWS provides a pre-defined IAM policy for ALB operations.
	2.	This policy is attached to an IAM role.
	3.	The Kubernetes service account for the controller is linked to this IAM role using OIDC (OpenID Connect).
	4.	This allows the pod to securely assume AWS permissions without storing credentials.


