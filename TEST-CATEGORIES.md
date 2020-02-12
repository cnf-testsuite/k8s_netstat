# Test Categories
## Compatability Tests
#### CNFs should work with any Certified Kubernetes product and any CNI-compatible network that meet their functionality requirements.  The CNF Conformance Suite validates this by:
1. K8s conformance testing
    * Run [Sonobuoy](https://github.com/cncf/k8s-conformance/blob/master/instructions.md) on the cluster
3. K8s API testing
    * Run [API snoop](https://github.com/cncf/apisnoop) on the cluster
    * Test Beta endpoint usage
    * Test for generally available endpoints
5. CNI Plugin testing
    * Test if CNI Plugin follows the [CNI specification](https://github.com/containernetworking/cni/blob/master/SPEC.md)

## Stateless Tests
#### The CNF conformance suite checks if state is stored in a [custom resource definition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) or a separate database (e.g. [etcd](https://github.com/etcd-io/etcd)) rather than requiring local storage.  It also checks to see if state is resilient to node failure by:
1. Reseting the container and checking to see if the CNF comes back up
2. Using upstream projects for chaos engineering (e.g [Litmus](//https://github.com/litmuschaos/litmus))
## Security Tests
#### CNF containers should be isolated from one another and the host.  The CNF Conformance suite uses tools like [Falco](https://github.com/falcosecurity/falco), [Sysdig Inspect](https://github.com/draios/sysdig-inspect) and [gVisor](https://github.com/google/gvisor) to:
1. Check if any containers are running in privileged mode
2. Check if there are any shells
3. Check if any protected directories or files are accessed

## Scaling Tests
#### The CNF conformance suite checks to see if CNFs support horizontal scaling (across multiple machines) and vertical scaling (between sizes of machines) by using the native K8s [kubectl](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#scaling-resources) command to:
1. Test increasing/decreasing capacity
2. Test small scall autoscaling with kubectl
3. Test large scale autoscaling with load test tools like [CNF Testbed](https://github.com/cncf/cnf-testbed)
4. Test if the CNF control layer responds to retries for failed communication? (e.g. using [Pumba](https://github.com/alexei-led/pumba) or [Blockade](https://github.com/worstcase/blockade) for network chaos and [Envoy](https://github.com/envoyproxy/envoy) for retries)

## Configuration and Lifecycle Tests

#### Configuration and lifecycle should be managed in a declarative manner, using [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/), [Operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/), or other [declarative interfaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#understanding-kubernetes-objects).  The Conformance suite checks this by:

* Testing is CNF installed using a [versioned](https://helm.sh/docs/topics/chart_best_practices/dependencies/#versions) Helm chart? 
* Checking for a liveness entry in the helm chart and is the container responsive to it after a reset (e.g. by checking the [helm chart entry](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/))?
*  Checking for a readiness entry in the helm chart and is the container responsive to it after a reset?
*  Can we start the pod/container without mounting a volume (e.g. using [helm configuration](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)) that has configuration files?
* Testing to see if we can start pods/containers and see that the application continues to perform(e.g. using [Litmus](https://github.com/litmuschaos/litmus))
*  Testing by reseting any child processes, and when the parent process is started, checking to see if those child processes are reaped (ie. monitoring processes with [Falco](https://github.com/falcosecurity/falco) or [sysdig-inspect](https://github.com/draios/sysdig-inspect))?
* Testing if the CNF perform a rolling update? (i.e. [kubectl rolling update](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/))

## Observability Tests
* Is fluentd installed in the cluster?
* Is there traffic to Fluentd?
* Is Jaeger installed in the cluster?
* Is there traffic to Jaeger?
* Is prometheus installed in the cluster and configured correctly (e.g. using promtool)?
* Is there traffic to prometheus? 

## Installable and Upgradeable
#### The CNF Conformance suite will check for usage of standard, in-band deployment tools such as Helm (version 3) charts:
* Does the install script use [Helm](https://github.com/helm/)
* Is the Helm chart valid (e.g. using the [helm linter](https://github.com/helm/chart-testing))?
* Can the CNF perform a rolling update? (i.e. [kubectl rolling update](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/))

## Hardware and Affinity support
#### The CNF container should access all hardware and schedule to specific worker nodes by using a device plugin.  The CNF Conformance suite checks this by:

* Checking if the CNF is accessing hardware in its configuration filesee
* Testing if the CNF accessess hardware directly during run-time? (e.g. accessing the host /dev or /proc from a mount)
* Testing if the CNF accessess hugepages directly instead of via [Kubernetes resources](https://github.com/cncf/cnf-testbed/blob/c4458634deca5e8ab73adf118eedde32904c8458/examples/use_case/external-packet-filtering-on-k8s-nsm-on-packet/gateway.yaml#L29)?
* Testing if the cnf testbed performance output shows adequate throughput and sessions using the [CNF Testbed](https://github.com/cncf/cnf-testbed) (vendor neutral) hardware environment.