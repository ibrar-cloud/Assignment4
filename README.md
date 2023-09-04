# Assignment4

Before we begin with the Project, we need to make sure we have the following prerequisites installed:

EC2 ( AMI- Ubuntu, Type- t2.medium )
Docker
Minikube
kubectl

Step 1: Clone the source code
using git clone command

Step 2: Containerize the Application using Docker
###Write a Dockerfile with the following code:
FROM node:19-alpine3.15
WORKDIR /reddit-clone
COPY . /reddit-clone
RUN npm install 
EXPOSE 3000
CMD ["npm","run","dev"]

Step 3) Building Docker-Image
docker build -t <DockerHub_Username>/<Imagename> .
Step 4) Push the Image To DockerHub
Now push this Docker Image to DockerHub so our Deployment file can pull this image & run the app in Kubernetes pods.
First login to your DockerHub account using Command i.e docker login and give your username & password.
Then use docker push <DockerHub_Username>/<Imagename> for pushing to the DockerHub.
You can use an existing docker image i.e trainwithshubham/reddit-clone for deployment.

Step 5) Write Deployment.yml file
Let's Create a Deployment File For our Application. Use the following code for the Deployment.yml file.
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
        image: trainwithshubham/reddit-clone
        ports:
        - containerPort: 3000

Step 6) Write Service.yml file
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

Step 7) Deploy the app to Kubernetes & Create a Service For It
Now, we have a deployment file for our app and we have a running Kubernetes cluster, we can deploy the app to Kubernetes. To deploy the app you need to run following the command: kubectl apply -f Deployment.yml Just Like this create a Service using kubectl apply -f Service.yml
If You want to check your deployment & Service use the command kubectl get deployment & kubectl get services


Step 8) Let's Configure Ingress
Let's write ingress.yml and put the following code in it:
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
Minikube doesn't enable ingress by default; we have to enable it first using minikube addons enable ingress command.
If you want to check the current setting for addons in minikube use minikube addons list command.
Now you can able to create ingress for your service. kubectl apply -f ingress.yml use this command to apply ingress settings.
Verify that the ingress resource is running correctly by using kubectl get ingress ingress-reddit-app command.



Step 9) Expose the app
First We need to expose our deployment so use kubectl expose deployment reddit-clone-deployment --type=NodePort command.
You can test your deployment using curl -L http://192.168.49.2:31000. 192.168.49.2 is a minikube ip & Port 31000 is defined in Service.yml
Then We have to expose our app service kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &
