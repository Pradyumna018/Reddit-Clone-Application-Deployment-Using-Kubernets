# Node-Application-Deployment-Using-Kubernets

Before we begin with the Project, we need to make sure we have the following prerequisites installed:

1- EC2 ( AMI- Ubuntu, Type- t2.medium )

2- Docker

3- Minikube

4- kubectl

You can Install all this by doing the below steps one by one. and these steps are for Ubuntu AMI.

# For Docker Installation
```
sudo apt-get update

sudo apt-get install docker.io -y
```
```
sudo usermod -aG docker $USER && newgrp docker
```
# For Minikube & Kubectl
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube 
```
```
sudo snap install kubectl --classic

minikube start --driver=docker
```

### Great! You're all set for the project. Your Minikube cluster is now prepared for deploying the Reddit clone application. 


# Step 1: Clone the source code

The first step is to clone the source code for the app. You can do this by using this command :
```
git clone https://github.com/Pradyumna018/Node-Application-Deployment-Using-Kubernets.git
```
# Step 2: Containerize the Application using Docker

Write a Dockerfile with the following code:

```
FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone

RUN npm install 

EXPOSE 3000

CMD ["npm","run","dev"]

```
# Step 3) Building Docker-Image

Now it's time to build Docker Image from this Dockerfile. 

``docker build -t <DockerHub_Username>/<Imagename>``   use this command to build a docker image

# Step 4) Push the Image To DockerHub

Now push this Docker Image to DockerHub so our Deployment file can pull this image & run the app in Kubernetes pods.

First login to your DockerHub account using Command i.e __docker login and give your username & password.__

Then use **docker push <DockerHub_Username>/<Imagename>** for pushing to the DockerHub.

You can use an existing docker image i.e pradyumnam/reddit-clone for deployment.


# Step 5) Write a Kubernetes Manifest File

Now, You might be wondering what this manifest file in Kubernetes is. Don't worry, I'll tell you in brief.

When you are going to deploy to Kubernetes or create Kubernetes resources like a pod, replica-set, config map, secret, deployment, etc, you need to write a file called manifest that describes that object and its attributes either in yaml or json. Just like you do in the ansible playbook.

Of course, you can create those objects by using just the command line, but the recommended way is to write a file so that you can version control it and use it in a repeatable way.


# Write Deployment.yml file

Let's Create a Deployment File For our Application. Use the following code for the Deployment.yml file.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: pradyumnam/reddit-clone
        ports:
        - containerPort: 3000
```

# Write Service.yml file

```
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000
  selector:
    app: reddit-clone
```

# Step 6) Deploy the app to Kubernetes & Create a Service For It

Now, we have a deployment file for our app and we have a running Kubernetes cluster, we can deploy the app to Kubernetes. 

To deploy the app you need to run following the command: ``kubectl apply -f Deployment.yml``

Just Like this create a Service using : ``kubectl apply -f Service.yml``

If You want to check your deployment & Service use the command : __kubectl get deployment__ & __kubectl get services__


# Step 7) Expose the app

First We need to expose our deployment so use command :  ``kubectl expose deployment reddit-clone-deployment --type=NodePort``

You can test your deployment using __curl -L http://192.168.49.2:31000. 

192.168.49.2__ is a minikube ip & Port 31000 is defined in Service.yml

Then We have to expose our app service : ``kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &``


# Step 8) Let's Configure Ingress

Let's write ingress.yml and put the following code in it:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
```

Minikube doesn't enable ingress by default; we have to enable it first using ```minikube addons enable ingress``` command.

If you want to check the current setting for addons in minikube use ```minikube addons list``` command.

Now you can able to create ingress for your service. ```kubectl apply -f ingress.yml``` use this command to apply ingress settings.

Verify that the ingress resource is running correctly by using ```kubectl get ingress ingress-reddit-app``` command.

# Test Ingress

Now It's time to test your ingress so use the __curl -L domain/test__ command in the terminal.

# OUTPUT 


![fg](https://github.com/Pradyumna018/Reddit-Clone-Application-Deployment-Using-Kubernets/assets/136186419/95dbd28d-2db4-4711-9173-6365c531064d)





