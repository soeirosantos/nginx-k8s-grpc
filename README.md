# Ngnix IngressController with gRPC

In this guide we are going:

* Configure nginx IngressController on GKE (GCP kubernetes service)
* Deploy a gRPC service behind the nginx IngressContoller
* Test the gRPC service using a gRPC client

The content of this guide is based on the following material:
* https://www.nginx.com/blog/nginx-1-13-10-grpc/
* https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/installation.md
* https://grpc.io/docs/quickstart/go.html

Let's get started from the very early stage in case you don't have the environment ready.

- Install the gcloud CLI
https://cloud.google.com/sdk/docs/quickstart-macos

* Initialize the environment

```bash
$ gcloud init
```

* Install kubectl

```bash 
$ gcloud components install kubectl
```

* Configure some useful environment variables (change these values accordingly what makes sense for you)

```bash
$ export MY_REGION=us-east1 \
export MY_ZONE=us-east1-b \
export CLUSTER_NAME=cluster_name \
export PROJECT_ID=project_id #this ID must be the same as your project on GCP
export SERVICE_ACCOUNT=service-account #for the purpose of this guide it could be the email address you use to access GCP
```

* Configure gcloud

```bash
$ gcloud config set project $PROJECT_ID \
gcloud config set compute/region $MY_REGION \
gcloud config set compute/zone $MY_ZONE
```

* Check your configuration

```bash
$ gcloud config list
```

* Create the cluster

If it's the first time you are using GKE in this project ID then you'll need to activate "Kubernetes Engine API"

```bash
$ gcloud container clusters create $CLUSTER_NAME --num-nodes 3
```

* Get the credentials to work with `kubectl`

```bash
$ gcloud container clusters get-credentials $CLUSTER_NAME --zone $MY_ZONE --project $PROJECT_ID
```

* Now, lets start configuring the IngressController. Clone this repository:

```bash
$ git clone https://github.com/soeirosantos/nginx-k8s-grpc.git | cd nginx-k8s-grpc
```

```bash
$ kubectl apply -f ./ingress-install/common/ns-and-sa.yaml
$ kubectl apply -f ./ingress-install/common/default-server-secret.yaml # self signed certificate
$ kubectl apply -f ./ingress-install/common/nginx-config.yaml # http2: True
```

* Bind admin credentials to your account

```bash
$ kubectl create clusterrolebinding $SERVICE_ACCOUNT-cluster-admin-binding --clusterrole=cluster-admin --user=$SERVICE_ACCOUNT
```

```bash
$ kubectl apply -f ./ingress-install/rbac/rbac.yaml
```

* Deploy the nginx IngressController

```bash
$ kubectl apply -f ./ingress-install/deployment/nginx-ingress.yaml
```

* Check if it's running

```bash
$ kubectl get pods --namespace=nginx-ingress
```

* Create the service to get access to the IngressController

```bash
$ kubectl apply -f ./ingress-install/service/loadbalancer.yaml
```

* Get the External IP address

```bash
$ kubectl get svc nginx-ingress --namespace=nginx-ingress # repeat until the external ip is set
```

* Edit the file `/etc/hosts` file and add this IP to the end of the file using this host: `grpc.example.com`

- Now we are going to deploy the gRPC service and expose it through the nginx Ingress.

```bash
$ kubectl create -f grpc-hello.yaml
$ kubectl create -f grpc-ingress-tls.yaml
```

* Now run the golang gRPC client to test the configuration

```bash
$ go run grpc/greeter_client/main.go
```

If you have question about how to run the Golang gRPC client, access this page: https://grpc.io/docs/quickstart/go.html