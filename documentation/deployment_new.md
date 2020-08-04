# Deployment instructions for TESK
## Requirements
*  **A Kubernetes cluster version 1.9 and later**. TESK works well in multi-tenancy clusters such as OpenShift and requires access to only one namespace. 
* **A default storage class.** To handle tasks with I/O TESK always requires a default storage class (regardless of the chosen storage backend). TESK uses the class to create temporary PVCs. It should be enough that the storage supports the RWO mode.
* **A storage backend** to exchange I/O with the external world. At the moment TESK supports:
  * FTP. R/W access to a single FTP account. 
  * Shared file system. This usually comes in the form of a RWX PVC.
  * S3 (WiP). R/W access to one bucket in S3-like storage

## Installing TESK
### Helm
### Jinja templates

### Exposing TESK API

### Storage backends

#### Shared file system
#### FTP
#### S3


