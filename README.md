# KinD
Kind, stands for Kubernetes in Docker. It's a lightweight alternative to MiniKube
which is based on Virtual Machines.  
For more information visit [Kind](https://kind.sigs.k8s.io/).

## Creating a cluster
```sh
kind create cluster  --name=<cluster-name>
```

## Managing contexts
Creating a cluster will modify our ~/.kube/config file. This file is responsible for
authentication of different contexts. In order to switch between contexts:

```sh
kubectl config get-contexts  # list all contexts
kubectl config use-context <context-name>
```

## Deploying an application
Once we have our context properly configured, we can deploy our application using
`kubectl` commands.


## Accessing the application
### Linux
On linux, since containers run natively on the host, we can access the application in
a much simpler way, with no extra configuration.  
To access a NodePort service within the cluster, we can use the 2 following commands,
to get the address of the service:

```sh
kubectl get nodes -o wide  # Get node's internal IP address
kubectl get svc  # Get NodePort Service port
```

The service will be accessible at the following address: `<node-ip>:<svc-port>`

### Windows
On Windows, things are a little bit different. Since the cluster itself is a Docker
container, we have to expose the ports of the services to the outside world in the
container.  
The easiest way to do this is to create the cluster from a template like this:
```yaml
# cluster-config.yml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
    protocol: TCP
```
Then we can create the cluster using the following command, which will expose the port 30000 on the container:
```sh
kind create cluster --config=cluster-config.yaml --name=<cluster-name>
```
Since the cluster or Docker container exposes port 30000 to the outside, we can
map the service to the port by using the same port:
```yaml
# node-port-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  ports:
    - targetPort: 8008
      port: 80
      nodePort: 30000  # same port as exposed in the cluster
      protocol: TCP
  selector:
    app: catalog
```
Then we will be able to access the service from `http://localhost:30000`.  
Putting all this together, we can create a cluster and deploy a simple application with:
```sh
kind create cluster --config=cluster-config.yaml --name=e2eo
kubectl apply -f registry.yaml
kubectl apply -f catalog-deploy.yaml
kubectl apply -f node-port-svc.yaml
# access the service from http://localhost:30000
```