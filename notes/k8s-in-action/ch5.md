# Services

Pods need a way to find other pods if they want to consume APIs that other pods provide. Unfortunately you can't rely on the IP address of the pod directly to communicate with it for the following reasons:
* Pods are ephemeral
* K8s assigns IP address to the pods after they are scheduled and before they are started.
* Horizontal scaling of pods means there are duplicate pods providing the same service.

In order to solve the above challenges K8s provides a resource type called as `Service`

> Service has a IP address and port that never change while it exists. Clients can open connnections to that IP and port and those connections are then routed to one of the pods backing that service. As a result the backing pods can have any IP address and can be moved around the cluster at any time.

## Creation of Services

* Service make use of label selector to provide a service endpoint to a group of pods identified by the label.
* Labels selectors for services are defined in json or yaml files using maps, and only equality-based requirement selectors are support ( no set-based selectors). 
  * e.g  
  ```yaml
  selector:
    app: nodejs
  ```  
  Note that `matchLabels` is not used.
* The Service requires a `qualified DNS name` , `port` and the `targetPort` ( container port the serivice will forward to).
* Service can have `sessionAffinity` to a particular pod based on the client IP addr  
* If your pods expose multiple ports you can tie them up using a single service as you can configure multiple ports in a service.
* If you name your ports in your container spec of any resource manifest like pod/deployment/replicasets , these names act as aliases to the port number. You can then use these names/aliases within the service spec `targetPort`. The main benefit of this is that next time if you change the container port number you don't have to reconfigure the service. 
* The address of the service will remain unchanged throughout the lifetime of the service. If the service is recreated it will get a new address.

## Discovering services

* All the services' IP addr and ports within your cluster are exposed via environment variables to each container within each pod in the entire cluster.
* The environment variables for the following naming schemes :   
    <service_name>\_SERVICE_PORT\_<port_name_in_service_manifest> = service port number   
    <service_name>\_SERVICE_HOST\_ = service ip address 
* Make sure that _ALL_ the ports are named in the container spec as well as in the service spec as a best practice convention.
> Dashes in the service name are converted to underscores and all letters are uppercased when the service name is used as the prefix in the environment variable’s name
> 


## kubectl commands

`kubectl exec <pod_name> -c <container_name> -- <any_command that should run within the container >`  
`kubectly apply -f <k8s_obj>` - configures or creates the object