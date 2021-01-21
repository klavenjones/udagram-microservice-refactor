# README

# Udacity Udagram Microservice

The project is split into three parts:

1. [The Simple Frontend](/udagram-frontend) A basic Ionic client web application which consumes the RestAPI Backend.
2. [The RestAPI Feed Backend](/udagram-feed-api), a Node-Express feed microservice.
3. [The RestAPI User Backend](/udagram-user-api), a Node-Express user microservice.

## Links

- Docker hub images found [here](https://hub.docker.com/u/klavenj)
- Screenshots are documented and found in the [screenshots](/screenshots) directory

## Docker Build Steps

As follows:

```
cd udagram-deployment/dockerdocker-compose -f docker-compose-build.yaml build --paralell
```

To push images:

```
docker-compose -f docker-compose-build.yaml push
```

Ensure you are logged into docker hub first.

## Deployment

### Cloud Deployment

Ensure the following requirements are met:

- aws cli installed
- aws eksctl installed
- kubectl installed

Create the aws cluster and save the config for later:

```
cd /udagram-deployment/ekseksctl create cluster -f udagram-cluster.yaml --kubeconfig=./udagram-config.yaml
```

Setting KUBECONFIG to the current path ensures we pick up this config:

```
export KUBECONFIG=$(pwd)/udagram-config.yaml
```

The cluster can be destroyed by:

```
cd /udagram-deployment/ekseksctl delete cluster -f udagram-config.yaml
```

Next the services can be deployed as follows:

```bash
cd /udagram-deployment/k8s
kubectl apply -f aws-secret.yaml
kubectl apply -f env-secret.yaml
kubectl apply -f env-configmap.yaml
kubectl apply -f backend-feed-deployment.yaml
kubectl apply -f backend-feed-service.yaml
kubectl apply -f backend-user-deployment.yaml
kubectl apply -f backend-user-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f reverseproxy-deployment.yaml
kubectl apply -f reverseproxy-service.yaml
```

Check the deployment:

```
kubectl get pod 
kubectl get deployment
```

To test the deployment locally:

```
kubectl port-forward service/frontend 8100:8100
kubectl port-forward service/reverseproxy 8080:8080
```

Open localhost:8100 in a browser to check the docker deployment.

### Local Deployment

The project can be deployed and tested locally using the docker-compose file under /udagram-deployment/docker as follows:

Set the variables in the /udagram-deployment/docker/.env file. These are:

```
POSTGRESS_USERNAME=
POSTGRESS_PASSWORD=
POSTGRESS_DB=
POSTGRESS_HOST=
AWS_REGION=
AWS_PROFILE=
AWS_MEDIA_BUCKET=
JWT_SECRET=
```

Now run the system:

```
cd /udagram-deployment/docker/docker-compose up -d
```

Open localhost:8100 in a browser to check the docker deployment.

## Travis CI/CD

Travis has been configured to carry out the following CI/CD operations:

- Build the docker images
- Push the docker images to Docker Hub

### Rolling Updates

Implemeted in the /udagram-deployment/k8s deployment files.