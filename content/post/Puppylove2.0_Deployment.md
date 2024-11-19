---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Puppylove2.0 Deployment"
subtitle: ""
summary: "Using kubernetes to deploy a go backend and postgres application"
authors: [Pratham Sahu]
tags: []
categories: []
date: 2024-02-04T00:06:40+05:30
lastmod: 2024-02-04T00:06:40+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
I was recently deciding on a platform to host Puppylove2.0, our latest build at PClub. I had a few things in mind when I was making this decision:

- Reproducibility
- Consistency
- Failure aversion/Availability
- Metrics
- Versioning

After a bit of research, I decided to use Kubernetes for hosting the backend and database on the Google Kubernetes Engine(Owing to the 300$ of free credits).

Kubernetes ensures reproducibility using .yaml files that can spawn up clusters in no time. The images used were hosted on dockerhub. The database could also self-heal or restart at the same state on crashes owing to the Persistant Volume feature. The "pods" hosting the backend could be easily exposed using Kubernetes service making it very easy to use. The metrics given by the GKE also seem pretty neat.

One of the major reasons for choosing this was also that I could deploy both the backend and db on the same cluster reducing latency of DB calls. Also the backend could be easily scaled by spawning more replicas. 

Not knowing a thing about Kubernetes, I spent quite a few days familiarising with hosting a backend and a persistant database on it. I am writing this blog to make the process easier for people.

## Hosting the Backend
Our backend was written using the go-gin framework. We first built a docker image which was easily deployed on K8.
### ConfigMaps for env variables:

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume. Since my backend was using environment variables and I wanted them to flexible, I used ConfigMaps.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-core-config
data:
  POSTGRES_HOST : "db-core"
  POSTGRES_PORT : "5432"
  POSTGRES_DB : "puppylove"
  POSTGRES_USER : "postgres"
  ADMIN_ID : "admin1"
  ADMIN_PASS : "admin2"
  DOMAIN : "localhost"
  EMAIL_ID : "hello@iitk.ac.in"
  GIN_MODE : "release"
```

### Deploying backend pods:
To deploy the backend, we can write a simplistic yaml file, which refers to the ConfigMap for environment variables.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-core-deployment
  labels:
    app: backend-core
spec:
  selector:
    matchLabels:
      app: backend-core
  template:
    metadata:
      labels:
        app: backend-core
    spec:
      containers:
        - name: backend-core
          image: prathamsahu52/backend_puppylove:1.0.6
          ports:
            - containerPort: 8000
          envFrom:
          - configMapRef:
              name: backend-core-config
```
## Hosting the Database:
We needed our Postgres database to be completely failure averse as crashes and permanent data loss were not an option for a 7 day event. Databases in kubernetes can have 'Persistant Volumes', that get new databases up and running with consistent data, if it crashes.

### Setting up persistant volumes

First we create storage classes to define the type of storage being offered. You can read more about it [here](https://kubernetes.io/docs/concepts/storage/storage-classes/)

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gold
provisioner: kubernetes.io/gce-pd
volumeBindingMode: Immediate
allowVolumeExpansion: true
reclaimPolicy: Delete
parameters:
  type: pd-standard
  fstype: ext4
  replication-type: none
```

After setting it up, we spawn up persistant volumes that will ensure consistency in data even when the database crashes. [You can read it here](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-core-pvc
spec:
  storageClassName: gold
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

```
### Deploying the Backend Pods:
Now we can deploy the backend pods while keeping db-core-pvc as the persistant volume claim to ensure durability of data. We also change the ConfigMap of the backend to match this deployment.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-core-deployment
  labels:
    app: db-core
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db-core
  template:
    metadata:
      labels:
        app: db-core
    spec:
      volumes:
        - name: db-core-persistent-storage
          persistentVolumeClaim:
            claimName: db-core-pvc    
      containers:
        - name: db-core
          image: postgres
          volumeMounts:
            - name: db-core-persistent-storage
              mountPath: /var/lib/postgresql/puppylove_data
          env:
          - name: POSTGRES_DB
            value: "puppylove"
          - name: POSTGRES_USER
            value: "postgres"
          - name: POSTGRES_PASSWORD
            value: "postgres"
          ports:
            - containerPort: 5432
```
## Exposing the backend and Database:
I did not expose my database, but only my backend as my backend could query my database through internal IP. This adds another layer of security to the application.

We make use of [Service](https://kubernetes.io/docs/concepts/services-networking/service/) to expose our application.

The yaml file is mentioned here:
```
apiVersion: v1
kind: Service
metadata:
  name: backend-core
spec:
  ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
      nodePort: 30100
  selector:
    app: backend-core
  sessionAffinity: None
  type: NodePort
```

While exposing you also need to change the firewall rules to allow traffic on a given port. They can be found [here](https://cloud.google.com/kubernetes-engine/docs/how-to/exposing-apps#creating_a_service_of_type_nodeport)


To understand how to actually deploy them on gcp, the [DEPLOYMENT.md] (https://github.com/pclubiitk/puppylove2.0_backend/blob/main/DEPLOYMENT.md) written by me to deploy the application should be helpful.

## Ingress 
<u>Note</u>: This section was contributed by my friend [Yash](https://www.linkedin.com/in/yash-pratap-singh-284b122b2/)

Finally, we require an HTTPS load balancer to efficiently direct all incoming requests to the exposed backend, serving two main purposes:
1. Providing a secure IP for HTTPS requests.
2. Managing the overall system load.

In a Kubernetes Cluster, the tool for load balancing is called Ingress. Ingress, as an API object, manages external access to services in the cluster. It sets up rules for routing traffic to the correct backend services. The rules in Ingress specify where a request should go within the cluster. The actual implementation of Ingress is through an Ingress Controller, with GKE having its own easy-to-set-up and configurable Ingress Controller. (There are other Ingress controllers like Nginx ingress controller)

To establish HTTPS Ingress for the service, an SSL certificate is required. Connecting it to a domain name makes the endpoint secure by all browser standards.
#### By GKE Console
When setting up HTTPS Ingress, we need an SSL certificate. You can upload your SSL certificate, or if you have a registered domain, you can use Google managed Certificates. In our case, we had a domain in Google Domains. Creating a subdomain, we used it when setting up the Ingress. Once the Ingress is created, it provides an external IP. We then point our DNS to this IP. This step verifies Google's managed SSL, and the subdomain directs to the Ingress service, forwarding all requests to the exposed backend service.
#### Ingress.yaml
You can set up an Ingress on your GKE instance by applying an Ingress file also, but make sure you've got an SSL certificate ready beforehand.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: "gce"
    networking.gke.io/pre-shared-cert: "[NAME]"
spec:
  rules:
  - host: your-domain.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: your-backend-service
            port:
              number: 80
  tls:
  - hosts:
    - your-domain.com
    secretName: "[SECRET_NAME]"
```

## Scalability
One of the biggest advantages of kubernetes is that it is highly scalable. Replicas can be spawned by changing a few lines of code. However, since we use it for a campus of around 10000 people, more than 1 pod was not required, so we stuck to it.

### Other References:
- https://cloud.google.com/blog/products/containers-kubernetes/exposing-services-on-gke
- https://cloud.google.com/kubernetes-engine/docs/how-to/exposing-apps
