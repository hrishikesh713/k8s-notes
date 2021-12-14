# Replication   

standalone pods require manual supervision , creation and management. In real-world you want your containers/apps to stay up and running and healthy automatically without any manual intervention. In order to accomplish that you require a different kind of resource such as `ReplicationControllers/ReplicaSets/DaemonSets` or `Deployments` which will create and manage the pods for you. These different type of resources accomplish different level of management of pods.  
When you create an unmanaged standalone pod a cluster node is selected to run the pod and then its containers are run on that node. K8s will then monitor those containers within the pod and automatically restart them if they fail for some resond. But if the entire cluster node fails the pods on that node will not be replaced with new ones unless they are managed by the above mentioned resources.

## Keeping pods healthy
* `kubelet` is incharge of running the containers within the pod and keep them running as long as the pod exists. So out of the box deploying a pod will give your app the ability to automatically restart on failure/crash.
* But sometimes app may not crash but it won't be working as expected. E.g your app is catching a fatal error and has ended up in an error state. There needs to be a way for your app to signal k8s that it is not healthy.
* But the above strategy is also not full proof as your app may have got stuck in a deadlock and its impossible for your app to signal k8s to restart it. To make sure apps are restarted there must be a way to check apps health from outside.
* k8s provides a liveness probe which you can set in the pod's specification. There are three mechanisms to achieve this :
  * HTTP GET probe is performed by k8s over a route specified by the container within the pod. If k8s doesn't get any response or gets an error response that means the container is unhealthy.
  * TCP Socket probe opens a tcp conn to the specified port in the pod's spec. If connection is established successfully then container within the  pod is healthy.
  * Exec probe executes an arbitary command inside the container and check the command's exit status code. If exit code is non-zero then container within the pod is unhealthy.
* liveness Probe has three options : delay, timeout and period. Setting up delay is important as k8s will wait for X delay time before starting the probe.
* `kubectl describe pods <pod_name>` will give you the liveness details for each container. K8s will send a sigkill if the container is unhealthy. `kubectl get pods <pod_name>` will show cumulative restarts from all containers within that pod. The error code is addition of `128 + SIGKILL/SIGTERM` . So it can be either 137 or 148. 
* `SIGTERM` will restart the container `SIGKILL` will create a new container and won't restart the same container again.
* the liveness check is done by `kubelet` running on the cluster node and k8s control plane components running on the master nodes have no part in this process.
* If the node itself crashes the control plane must create replacements for all the pods that went down with the node. Unfortunately it doesn't do that with pods that are created directly as these directly created pods are managed by the kubelet and kubelet is local to the cluster node. So it also goes down when node goes down.
* Replicaset and Daemonset are in `apps/v1` api of k8s.

## Replication via ReplicaSets

* Replicaset contains label selector which help them to maintain the ownership of pods that match that label criteria.
* Its manifest also contains a template for the pod in case it has to create them to get the pod count to the desired state. Any other pod which may or may not match the template can be owned by the Replicaset if the label matches.
* the way that pods and replicasets are linked together is via the `metadata.ownerReferences` field in the pod manifest.
* replicaSets only care about labels on the pods to determine the replication policy
* `spec.template` is a required field for replicaset. This will be used to create pods with specific containers to meet the desired state. If the desired state is already achieved before creation of a replicaset , it won't attempt to reload the pods to match the template. As a result a rolling upgrade is not possible with a replicaset.
* the main difference between `replication controller` and `replica set` is that `replica set` have more expressive label selectors via `matchExpressions` property of `label Selectors`.
* You can control the pod placement on the nodes by using the `nodeSelector` property within the `template.spec` for replica set. Unfortunately you cannot control the number of pods per node matching the `nodeSelector` property.

## Controlling pod placement via DaemonSets

* If you want to run exactly one pod per node DaemonSets need to be used. All the scripts that would have been traditionally been started by systemd can be good candidates for DaemonSets.
* If a node goes down DaemonSets do not restart the pod on another node. But if a new node is added to the cluster the Daemonset will start a new pod on it.
* In order to limit the deployment of daemonsets pods you can use the `nodeSelector` property to deploy the pods only on specific nodes.
* Daemonsets completely bypass k8s scheduler hence any scheduler properties like `unschedulable` which prevents nodes to be scheduled on the node doesn't affect daemonset. It will still deploy pods on the nodes that are marked as `unschedulable` . 
* Daemonsets will terminate any unmanaged pods that are already running on the node that match the label selector.     
* If pods are managed by some controller then other controllers won't interfere with the pods even if the label matches. They will just create their own pods.

  > ReplicaSets take control of unmanaged pods with same labels. DaemonSets terminate unmanaged pods with same labels.



## Running pods that perform a single completable Task

* DaemonSets or ReplicaSets are made to run pods that execute continuous tasks. That means if a task exists the pod will exit and these resources will again run the pod to meet the desired state.
* If you want to run a completable task that shouldn't run again after its completed then you need to use a `Job Resource`.
*  Job do not have a label selector so it doesn't interfere with unmanaged pods or pods managed by other controllers.
*  The job resource is in `batch/v1` api version of k8s.
*  If the node goes down on which the batch  job pod is running the job controller will schedule it on another node.
*  Using `CronJob` resource we can schedule a job to run at a particular time. All the normal cron job configurations can be applied to this resource. 


## Kubectl commands
`kubectl label pod <pod_name> <key=value>` - add label to a pod.  
`kubectl label pod <pod_name> <key=value> --overwrite` - overwrite the label of a pod  
`kubectl get pods -L <label_key>` - get all pods matching the label key  
`kubectl describe rs <replica_set_name>` - get info about a replicaset  
`kubectl scale job/rc <job_name/replicaset_name> --replicas <number>` - scale up/down a deployed replicaset or a batch job  
