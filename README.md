# Deploy Docker, Kubernetes, NodeJS, Express on Google Cloud Platform

## 1. Install Docker in your machine

Go to the [docker website](https://www.docker.com) and download the Free edition of docker, called the "Community Edition".

Or for MacOs/Linux run this command :

```
$ brew install docker
```

## 2. Install Google Cloud SDK

Download [Google Cloud SDK.](https://cloud.google.com/sdk/docs/)
Once downloaded, extract the folder and run the following command to install gcloud and add it to your path.

```
$ ./google-cloud-sdk/install.sh
```

You can now authenticate gcloud with your Google account on your terminal by running the initialization command.

```
$ gcloud init
```

To check your current configurations, run the configuration list command.

```
$ gcloud config list
```


## 3. Install Kubernetes command

```
$ gcloud components install kubectl
```

If you need to install any other provisions of Google Cloud SDK, simply run the command 
```
$ gcloud components list
```

## 4. Create The GCP Project 

Go to the [Google Cloud Platform](https://console.cloud.google.com/) and create a new project.

## 5. Preparing our Deployment Environment

### a. Set default project

Once we have the gcloud and kubectl command-line tools installed, we can now set the project we created earlier as our default. We can easily do this with the config option.

```
$ gcloud config set project my-project-001122
```

### b. Create cluster

```
$ gcloud container clusters create my-project-cluster
```

The cluster may take some time to complete. Once it's done, again, set it as the default cluster.

```
$ gcloud config set container/cluster my-project-cluster
```

## 6. Create a Dockerfile
```
FROM node:9.11

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm install --only=production

# Bundle app source
COPY . .

EXPOSE 8000

CMD [ "npm", "start" ]
```

### a. Build a Dockerfile for Google Content Registry
```
docker build -t gcr.io/my-docker-203607/docker-app .
```

### b. Push a Docker image in Google Content Registry
```
gcloud docker -- push gcr.io/my-docker-203607/docker-app
```

## 7. Deployments: Instantiating A Container From the Docker Image

Create a file deployment.yml in your app project like this :

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: docker-deploy
  labels:
    #Project ID
    app: my-docker-203607
spec:
  #Run two instances of our application
  replicas: 2
  template:
    metadata:
      labels:
        app: my-docker-203607
    spec:
      #Container details
      containers:
        - name: docker-app
          image: gcr.io/my-docker-203607/docker-app
          imagePullPolicy: Always
          #Ports to expose
          ports:
          - containerPort: 8000
```

and run this command
```
kubectl create -f deployment.yml
```

Or run this command :

```
$ kubectl run my-project-deployment --image=gcr.io/${PROJECT_ID}/docker-app --port 8000
```

## 8. Expose application to External traffic

Create a file service.yml in your app project like this :
```
kind: Service
apiVersion: v1
metadata:
  #Service name
  name: docker-app-svc
spec:
  selector:
    app: my-docker-203607
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: LoadBalancer
```

and run this command
```
kubectl create -f service.yml
```

Or run this command :
```
$ kubectl expose deployment my-project-service --type=LoadBalancer --port 80 --target-port 8000
```

# Remove instance GCE

## 1. Remove Service
```
kubectl delete service my-project-service
```

## 2. Remove Deployment
```
kubectl delete service my-project-service
```

## 4. Check if all deleted
```
gcloud compute forwarding-rules list
```

## 3. Remove Container Cluster
```
gcloud container clusters delete my-project-cluster  --zone=europe-west2-b --async
```

## 4. Remove Docker images in Container registry
```
gcloud container images delete gcr.io/my-docker-203607/docker-app:latest
``` 


