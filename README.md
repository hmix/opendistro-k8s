# opendistro-k8s
Kubernetes configuration for evaluating [Open Distro](https://opendistro.github.io) on [Minikube](https://kubernetes.io/docs/setup/minikube).

## Preparing the infrastructure

No deployment script at the moment. To get the stuff running, my workflow is as follows.

1. Set env vars (I have a dedicated place for all my Minikubes)

```
export MINIKUBE_HOME=/home/$USER/vms/minikube
export PATH=$MINIKUBE_HOME/bin:$PATH
```

2. Configure and start Minikube instance

```
minikube --profile opendistro config set memory 7168
minikube --profile opendistro config set cpus 2
minikube --profile opendistro config set vm-driver kvm2
minikube --profile opendistro config set kubernetes-version v1.12.0
minikube start -p opendistro
```

3. Configure Docker to use the Minikube environment

```
minikube --profile opendistro docker-env
eval $(minikube --profile opendistro docker-env)
```

## Building Docker images

This is straight forward. The Dockerfiles use the upstream images from AWS and have been created for later customization.

```
docker build -t opendistro-for-elasticsearch:0.9.0 -f dev-minikube/Dockerfile-odfe .
docker build -t opendistro-for-elasticsearch-kibana:0.9.0 -f dev-minikube/Dockerfile-odfe-kibana .
```

## Deploying Kubernetes objects

ATM all objects for running Open Distro on Minikube reside in the folder, dev-minikube. Deploy them in this order.

```
kubectl apply -f ns.yml
kubectl apply -f sc.yml
kubectl apply -f pvc.yml
kubectl apply -f svc.yml
kubectl apply -f deployment-odfe.yml
kubectl apply -f deployment-odfe-kibana.yml
```

Note: Maybe you have to change permissions on the underlying folder for Elasticsearch data

```
minikube ssh
chown 1000:1000 /var/opendistro
```

## Access Elasticsearch and Kibana

- Elasticsearch

    ```
    curl -XGET https://$(minikube -p opendistro ip):30200 -u admin:admin --insecure
    curl -XGET https://$(minikube -p opendistro ip):30200/_cat/nodes?v -u -u admin:admin --insecure
    curl -XGET https://$(minikube -p opendistro ip):30200/_cat/plugins?v -u -u admin:admin --insecure
    ```

- Kibana

    Get Minikube IP address: `minikube -p opendistro ip`

    Point your Browser to http://$MINIKUBE_IP:30601

## Tear down the environment

To delete all Kubernetes objects simply delete the namespace: `kubectl delete ns opendistro`.
To delete the whole cluster, issue `minikube delete`.


