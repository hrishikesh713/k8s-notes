# Pods

## Concepts  

* Pod is a group of one or more co-located containers. A pod can never span multiple machines/nodes. They are like logical hosts with their own IP address.
* process A runs within a container and if that process needs to use IPC to communicate to another process B, then process B container needs to be co-located on the same machine. Pods make it easy to group such containers together.
* Pods are the higher level construct that binds such containers together.
* Pod is the smallest deployable unit in K8s . So if you have to deploy a container you need to wrap it with a pod.
* The containers in the pod represent processes that would have run on the same server/node/machine in a pre-container world.
* The containers within the same pod share the same Network, UTS ( you can change host name value within the namespace), IPC by default and PID namespace upon configuration. When sharing PID namespace the containers can see each other's processes and are able to [access each other's container's root directory](https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/).
* They can also share directories via k8s volume concept.
* Containers within the same pod have same port space and can communicate via localhost (loopback interface) because they share same network namespace. You will run into port conflicts if multiple containers/processes are using same port numbers.
* container logs can only be viewed/extracted for pods that in existence. So all the logs are ephemeral in nature and by default rotated.

## Flat inter-pod network  
* Every pod can connect to any other pod directly by using the other pod's IP address. There is no NAT (network address translation between them). As a result if you expose the POD's IP address externally you are basically allowing external clients to send without any preliminary security that NAT firewall normally provides.
* It doesn't matter if the pods are on different nodes , every pod is accessible via its IP address.
* This feat is achived by sotware defined network layered on top of actual network.

## Creation of Pods

### Parts of a Pod YAML Definition:  
* Metadata - name, namespace, labels, etc
* Spec - description of pod's contents like containers, volumes, etc.
* Status - description of the current condition of the pod . Status is only returned when you query the K8s API for the pod info. When creating a pod you don't provide status part.

## Questions

 > Why it is not a best practice to run multiple processes inside a single container?  

 * you need to provide your own process supervisor mechanism.
 * you need to make sure all processes are not logging to stdout. Very difficult to parse the logs.
 * When shutting down a container with multiple processes signal handling must be carefully implemented within the container and the docker daemon kill timeout also has to be made configurable for graceful shutdown.
 
 > When should I use multiple pods over multiple containers in a single pod for a multi-teir application?

 * Different scaling requirements - use multiple pods.
 * sidecar container pattern - use multiple containers in a pod.
 * components can run on different machines - use multiple pods.

> Why does containerPort config in the pod manifest only serve as informational purpose? 


## Must read articles :
* [understanding networking in K8s : pods](https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727)
* [understanding networking in k8s : services](https://medium.com/google-cloud/understanding-kubernetes-networking-services-f0cb48e4cc82)
* [networking basics](https://www.digitalocean.com/community/tutorials/an-introduction-to-networking-terminology-interfaces-and-protocols)
* [understanding ip addresses](https://www.digitalocean.com/community/tutorials/understanding-ip-addresses-subnets-and-cidr-notation-for-networking)
* [deep dive into linux namespaces](http://ifeanyi.co/)

## Kubectl commands

* `kubectl explain pods` - docs about pods
* `kubectl explain pods.spec` - docs about spec part of pod
* `kubectl create -f <manifest_file>` - creates a pod
* `kubectl get po <pod name> -o <json|yaml>` - gives full description in yml/json format
* `kubectl get pods` - get all pods and their statuses
* `kubectl logs <pod-name> -c <container-name>` - get container logs within the pod
* `kubectl port-forward <pod-name> <local-machine-port>:<container-exposed-port>` - for quickly sending request to container web service running within a pod