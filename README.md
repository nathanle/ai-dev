# AI Multi-User Development Platform 

# Installation

## Setup Environment

Copy the kubernetes cluster config to ~/.kube/config

## Envoy Gateway

The Envoy Gateway provides cluster ingress for Services running in Kubernetes.

### The following command will install the gateway:
```shell
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.7.0 -n envoy-gateway-system --create-namespace
```
### The following will install the GatewayClass and the Gateway definition itself:
```shell
kubectl apply -f gateway/quickstart.yaml
```

### Install ReferenceGrant.yaml
This manifest contains the grants to allow the gateway to forward traffic to services that exist in different namespaces

```shell
kubectl apply -f gateway/ReferenceGrant.yaml
```

### There is a tool for doing command line interactions specific to the envoy gateway. You can install this with your OS package manager:
```shell
brew install egctl
```
### Example use showing the status of the routes. In this example, everything is working except for ArgoCD as I have removed it from this cluster after defining the HTTPRoute and GRPCRoute.

```shell
# egctl x status all -A

NAME              TYPE       STATUS    REASON
gatewayclass/eg   Accepted   True      Accepted

NAMESPACE   NAME         TYPE         STATUS    REASON
default     gateway/eg   Programmed   True      Programmed
                         Accepted     True      Accepted

NAMESPACE   NAME                    PARENT       TYPE           STATUS    REASON
                                                 Accepted       True      Accepted
default     httproute/cpu           gateway/eg   ResolvedRefs   True      ResolvedRefs
                                                 Accepted       True      Accepted
default     httproute/gpu           gateway/eg   ResolvedRefs   True      ResolvedRefs
                                                 Accepted       True      Accepted
default     httproute/keycloak      gateway/eg   ResolvedRefs   True      ResolvedRefs
                                                 Accepted       True      Accepted


NAMESPACE   NAME                                              ANCESTOR REFERENCE   TYPE       STATUS    REASON
default     clienttrafficpolicy/enable-tcp-keepalive-policy   gateway/eg           Accepted   True      Accepted

NAMESPACE   NAME                     ANCESTOR REFERENCE   TYPE       STATUS    REASON
default     securitypolicy/rstudio   gateway/eg           Accepted   True      Accepted
default     securitypolicy/vsc       gateway/eg           Accepted   True      Accepted
```
### Additional Files in gateway Folder:

* ClientTrafficPolicy.yaml - Provides TCP settings at gateway. Also can be used to enable ProxyProtocol to see the user's connecting IP address.

* SecurityPolicy.yaml - If an application does not have the ability handle OAuth, you can define it here so that the Envoy Gateway can do it. Note the `clientSecret` reference to a secret that contains the key for the OAuth app. 

## Let's Encrypt

This will install the helm chart for Let's Encrypt
```shell
helm repo add jetstack https://charts.jetstack.io

helm install \
cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--version v1.19.2 \
--set crds.enabled=true \
--set config.enableGatewayAPI=true \
--set config.kind="ControllerConfiguration"
```
In the **gateway** directory, there are two files: lets-enc-staging.yaml and lets-enc-prod.yaml. These files install the prod and staging version of the Let's Encrypt ClusterIssuer. They referenced in the annotations for the Envyoy Gateway:

```yaml
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-staging
```
or (for production version)
```yaml
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
```

Apply each file by running:
```shell
kubectl apply -f gateway/lets-enc-staging.yaml

kubectl apply -f gateway/lets-enc-prod.yaml
```

## Shared Storage
```
helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/
helm install shared-fs nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner
"nfs-ganesha-server-and-external-provisioner" already exists with the same configuration, skipping
NAME: shared-fs
LAST DEPLOYED: Fri Mar 13 09:33:16 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
The NFS Provisioner service has now been installed.

A storage class named 'nfs' has now been created
and is available to provision dynamic volumes.

You can use this storageclass by creating a `PersistentVolumeClaim` with the
correct storageClassName attribute. For example:

    ---
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: test-dynamic-volume-claim
    spec:
      storageClassName: "nfs"
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 100Mi
```

## JupyterHub



We are installing one instance for CPU operations and one for GPU operations. Dask (see below) for distributed workloads has been added as well.

<img width="2940" height="1236" alt="Screenshot 2026-07-02 at 10 50 46 AM" src="https://github.com/user-attachments/assets/1303f6ec-89ca-4b91-af6b-9c46093e8866" />


<img width="2938" height="1312" alt="Screenshot 2026-07-02 at 10 48 53 AM" src="https://github.com/user-attachments/assets/8f7b86aa-48e0-40ff-a127-02583f8cc736" />


We will update the `ai-dev.yaml` file to specify the pool ID for the node pool for CPU VMs and GPU VMs.

```
  extraNodeAffinity:
    required:
      - matchExpressions:
          - key: "lke.linode.com/pool-id"
            operator: In
            values:
              - "845636"
```

In the section above, 845636 should be replaced with the pool ID for CPU VMs in cpu-config.yaml. The same section in gpu-config.yaml should be configured with the pool ID for the GPU pool ID.

Once this is done, you can use the helm commands below to activate the ai-dev environment.

```shell
helm repo add jupyterhub https://hub.jupyter.org/helm-chart/
helm repo update
helm upgrade --cleanup-on-fail \
  --install ai-dev jupyterhub/jupyterhub \
  --namespace ai-dev \
  --create-namespace \
  --version=4.3.2 \
  --values apps/jupyter/cpu-config.yaml
```

## NVIDIA support
```
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.17.1/deployments/static/nvidia-device-plugin.yml
```
This will install a Daemonset for NVIDIA support in the LKE nodes for Kubernetes.

More info background information can be found here: https://techdocs.akamai.com/cloud-computing/docs/gpus-on-lke

## Dask Gateway

Dask Gateway provides a secure, multi-tenant server for managing Dask clusters. It allows users to launch and use Dask clusters in a shared, centrally managed cluster environment, without requiring users to have direct access to the underlying cluster backend (e.g. Kubernetes, HPC Job queues, etc…).


### CPU cluster usage
<img width="2938" height="1556" alt="Screenshot 2026-07-02 at 10 12 58 AM" src="https://github.com/user-attachments/assets/f1862639-efaa-49fe-9964-15bac9339d20" />

### GPU cluster usage
<img width="2876" height="1705" alt="Screenshot 2026-07-02 at 10 14 29 AM" src="https://github.com/user-attachments/assets/1d3cf09f-97ef-4526-836b-18d02c1aecf3" />


## SFTPGo

Provides SFTP or SCP for uploading files to the shared NFS mount point. Can also be configured to use individual PVCs.

