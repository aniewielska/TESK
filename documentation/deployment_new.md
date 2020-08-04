# Deployment instructions for TESK
## Requirements
*  **A Kubernetes cluster version 1.9 and later**. TESK works well in multi-tenancy clusters such as OpenShift and requires access to only one namespace. 
* **A default storage class.** To handle tasks with I/O TESK always requires a default storage class (regardless of the chosen storage backend). TESK uses the class to create temporary PVCs. It should be enough that the storage supports the RWO mode.
* **A storage backend** to exchange I/O with the external world. At the moment TESK supports:
  * FTP. R/W access to a single FTP account. 
  * Shared file system. This usually comes in the form of a RWX PVC.
  * S3 (WiP). R/W access to one bucket in S3-like storage

## Installing TESK
The preferred way to install TESK is by using Helm.
Previously, we provided Helm-like templates that generated K8s yaml descriptors that in turn could be installed using `kubectl`. That method of installing TESK is now deprecated and may be removed in the future version of TESK. However, the Jinja templates and documentation are still available [in the repository](deployment.md), so you should still be able to use this method, if you have done so previously.
### Helm
TESK can be installed using Helm 3 (tested with v3.0.0) using [this chart](../charts/tesk). It is best to create a dedicated namespace for TESK, although for test or development clusters it is fine to use the `default` namespace.
The documentation of the chart gives a desciption of all configuration options and below the most common installation scenarios have been described.
TESK installation consists of a single API installed as a K8s deployment and exposed as a K8s service. Additionally, TESK API requires access to the K8s API in order to create K8s Jobs and PVCs. That is why the installation additionally creates objects such as service accounts, roles and role bindings.
The chart does not provide a way to install the default storage class and that needs to be done independently by the cluster administrator.

### Exposing TESK API

After executing the chart with the default values, TESK API will be installed, but will be accessible only inside the cluster. There is a number of options of exposing TESK externally and they depend on the type of the cluster.
#### NodePort and LoadBalancer
The most basic way of exposing TESK on self-managed clusters and a good option for development clusters such as Minikube is to use a NodePort type of service.
In the chart set the values:
```
service.type="NodePort"
## Any values in the range 30000-32767 is fine. 31567 is used as an example
service.node_port: 31567
```
After installing the chart TESK API should be accessible under the external IP of any of your cluster nodes and the node port. For minikube you can obtain the IP by running:
```
minikube ip
```
or open Swagger UI of TESK directly in the browser with:
```
minikube service tesk-api
```
You should be able to see an empty list of tasks by calling
```
http://external_IP_of_a_node:31567/v1/tasks

{
  "tasks" : [ ]
}

```
If your cluster is provided by a Cloud Provider, you may be able to use a LoadBalancer type of service. In the chart set the value:
```
service.type="LoadBalancer"
```
and consult [K8s documentation](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/) on how to find out the IP of your TESK service.
#### OpenShift
The chart handles eexposing TESK API as OpenShift Route. 
In the chart set the value:
`clusterType: openshift`
TESK API should be accessible under (Swagger UI):
`https://project-name.openshift-host-name`
and
`https://project-name.openshift-host-name/v1/tasks`
should return an empty list of tasks.
#### Ingress
Recommened way of exposing any public facing API from K8s cluster such as TESK API is to use Ingress. 
You need:
* an **Ingress Controller**. If your cluster does not have one, the TESK Helm chart provides a way to Install one particular controller (the section `deploy_ingress`), but a recommened approach is to install an up-to-date Ingress Controller of your choice independently.
* a **Hostname** - a DNS entry, where you will expose TESK. TESK can be installed at a subpath as well.
* an **Ingress Resource**, which will instruct the Controller where to expose TESK. The chart provides an Ingress Resource template. 
* A **TLS certificate** to serve TESK over https. You can obtain one from a certificate authority or automatically obtain one from [Let's Encrypt](https://letsencrypt.org/). The K8s way to do it is by installing [cert-manager](https://cert-manager.io/) and creating an ACME Issuer.
The example values for TESK Helm chart to create Ingress Resource with annotations for cert-manager, but not to install the controller:
```
host_name: tes.ebi.ac.uk
ingress:
    active: true
    tls_secret_name: tes-ebi-ac-uk-tls
    tls_letsencrypt:
       annotations:
           kubernetes.io/ingress.class: nginx
           kubernetes.io/tls-acme: "true"
           cert-manager.io/issuer: letsencrypt-production
    deploy_ingress:
        active: false
```
List of tasks should be reachable under this URL:
```
https://tes.ebi.ac.uk/v1/tasks
```

### Storage backends
Documentation in Progress

#### Shared file system
#### FTP
#### S3


