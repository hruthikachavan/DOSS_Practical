
# Practical 5: Kubernetes Deployment and Scaling

## Aim
To deploy a containerized Node.js application using Kubernetes, expose it through a NodePort Service, and demonstrate horizontal scaling by increasing the number of application replicas.

## Objectives
- Start and verify a local Kubernetes cluster.
- Build a Docker image for the Node.js application.
- Deploy the application using a Kubernetes Deployment.
- Expose the application using a Kubernetes Service.
- Verify Pods, Deployments, and Services.
- Scale the application from two replicas to three replicas.

## Tools and Technologies Used
- **Docker Desktop:** Provides the container runtime.
- **Minikube:** Runs a local Kubernetes cluster.
- **kubectl:** Command-line tool for interacting with Kubernetes.
- **Kubernetes:** Orchestrates containers and manages application replicas.
- **Node.js:** Runtime used to execute the application.
- **VS Code:** Used to create and edit Kubernetes YAML files.

## Introduction
Kubernetes is an open-source container orchestration platform used to deploy, manage, and scale containerized applications.

In this practical, a Node.js application is packaged as a Docker image and deployed to a local Kubernetes cluster using Minikube. A Deployment manages two application replicas, while a NodePort Service exposes the application externally. Finally, the Deployment is scaled to three replicas to demonstrate horizontal scaling.

## Procedure

### Step 1: Start the Kubernetes Cluster

Command:

    minikube start --driver=docker

**Purpose:**
- Starts a local Kubernetes cluster using Minikube.
- Uses Docker as the driver to run the Kubernetes node inside a Docker container.
- Initializes the Kubernetes control plane and required components.

### Step 2: Verify the Kubernetes Node

Command:

    kubectl get nodes

**Purpose:**
- Displays the nodes registered with the Kubernetes cluster.
- Shows each node's status, role, age, and Kubernetes version.
- The node should show `Ready` before deploying the application.

### Step 3: Check Cluster Status

Command:

    minikube status

**Purpose:**
- Displays the status of the Minikube host, kubelet, and API server.
- Helps verify whether the Kubernetes cluster is running correctly.

### Step 4: Build the Application Image

Run this command from the Node.js project directory containing the Dockerfile:

    minikube image build -t nodejs-practical-app:1.0 .

**Purpose:**
- Builds a Docker image from the application's Dockerfile.
- Makes the image available to the Minikube cluster.
- `-t` assigns the image name and tag `nodejs-practical-app:1.0`.
- `.` specifies the current directory as the build context.

Verify the image:

    minikube image ls

This command lists images available inside Minikube.

### Step 5: Create the Deployment

Create `deployment.yaml` with the following configuration:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nodejs-app
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nodejs-app
      template:
        metadata:
          labels:
            app: nodejs-app
        spec:
          containers:
            - name: nodejs-container
              image: nodejs-practical-app:1.0
              imagePullPolicy: IfNotPresent
              ports:
                - containerPort: 5000

**Purpose of the Deployment:**
- Creates and manages the Node.js application Pods.
- `replicas: 2` requests two copies of the application.
- The selector identifies Pods labelled `app: nodejs-app`.
- The container uses the image built in Step 4.
- `containerPort: 5000` documents the port used by the application inside the container.
- `imagePullPolicy: IfNotPresent` allows Kubernetes to use the image already available locally.

### Step 6: Create the Service

Create `service.yaml` with the following configuration:

    apiVersion: v1
    kind: Service
    metadata:
      name: nodejs-service
    spec:
      type: NodePort
      selector:
        app: nodejs-app
      ports:
        - port: 5000
          targetPort: 5000
          nodePort: 30080

**Purpose of the Service:**
- Exposes the application Pods through a stable network endpoint.
- `type: NodePort` makes the Service accessible through a port on the Kubernetes node.
- `port: 5000` is the port exposed by the Service.
- `targetPort: 5000` forwards traffic to the application container.
- `nodePort: 30080` specifies the external NodePort.
- The selector routes traffic to Pods labelled `app: nodejs-app`.

### Step 7: Deploy the Application

Apply the Deployment:

    kubectl apply -f deployment.yaml

Apply the Service:

    kubectl apply -f service.yaml

**Purpose:**
- `kubectl apply` creates or updates Kubernetes resources from configuration files.
- `-f` specifies the YAML file containing the resource definition.
- The Deployment starts managing the application replicas.
- The Service provides network access to the application.

### Step 8: Verify the Deployment

Check Deployments:

    kubectl get deployments

**Purpose:** Displays the desired, current, and ready replica counts for each Deployment.

Check Pods:

    kubectl get pods

**Purpose:** Displays the application Pods and their readiness and status. Both Pods should eventually show `Running` and `1/1` ready.

Check Services:

    kubectl get services

**Purpose:** Displays the configured Services, their types, cluster IP addresses, and exposed ports.

Inspect application logs if needed:

    kubectl logs deployment/nodejs-app

**Purpose:** Displays logs from an application Pod selected through the Deployment, helping diagnose application errors.

### Step 9: Access the Application

Command:

    minikube service nodejs-service

**Purpose:**
- Finds the URL used to access the NodePort Service.
- Opens the application in a browser when supported by the environment.
- Allows verification that the Node.js application responds to requests.

The application must listen on port `5000` inside the container and bind to `0.0.0.0` to accept connections through the Kubernetes Service.

### Step 10: Scale the Application

Command:

    kubectl scale deployment nodejs-app --replicas=3

**Purpose:**
- Changes the desired number of application replicas from two to three.
- Kubernetes creates another Pod to match the new desired replica count.

Verify scaling:

    kubectl get deployments
    kubectl get pods

**Expected result:** The Deployment should show three ready replicas, and three application Pods should be running.

## Kubernetes Concepts Demonstrated

| Concept | Description |
|---|---|
| Cluster | A Kubernetes environment that manages containerized applications. |
| Node | A machine that runs Kubernetes workloads. |
| Pod | The smallest deployable unit in Kubernetes, containing one or more containers. |
| Deployment | Manages application replicas and supports scaling and updates. |
| Service | Provides a stable network endpoint for accessing Pods. |
| NodePort | Exposes a Service through a port on the Kubernetes node. |
| Replica | A running copy of an application Pod. |
| Horizontal scaling | Increasing or decreasing the number of application replicas. |
| kubectl | Command-line interface for managing Kubernetes resources. |
| YAML | Configuration format used to describe Kubernetes resources. |

## Screenshots

### 1. Installed Tool Versions
![Installed Versions](screenshots/tools-version.png)

### 2. Kubernetes Cluster
![Cluster Ready](screenshots/cluster-ready.png)

### 3. Application Image
![Application Image](screenshots/application-image.png)

### 4. Deployment Configuration
![Deployment YAML](screenshots/deployment-yaml.png)

### 5. Service Configuration
![Service YAML](screenshots/service-yaml.png)

### 6. Deployment Status
![Deployment Running](screenshots/deployment-running.png)

### 7. Application in Browser
![Application Browser](screenshots/browser-app.png)

### 8. Scaling to Three Replicas
![Scaling](screenshots/scaling.png)

## Result
The Node.js application was deployed on a local Kubernetes cluster using Minikube. A Deployment managed two replicas, a NodePort Service exposed the application on port 30080, and horizontal scaling was demonstrated by increasing the desired replica count to three.

## Learning Outcomes
- Learned to start and inspect a Kubernetes cluster.
- Learned to build an application image for Minikube.
- Understood Kubernetes Deployments, Pods, and Services.
- Learned to expose an application using NodePort.
- Practiced deploying resources using YAML files and kubectl.
- Demonstrated horizontal scaling of an application.
- Learned to verify application status and troubleshoot using logs.

## Conclusion
This practical demonstrated how Kubernetes can deploy, expose, and scale a containerized Node.js application. Minikube provided the local Kubernetes environment, while Deployments and Services were used to manage application replicas and network access.
