# hackfest

## Prerequisites

* RHEL-family machine (Fedora 35, Centos 8, RHEL 8)
  - VM should suffice too. We recommend a minimum of 2vCPUs and 8GB of RAM. If using Amazon EC2 instance, we recommend a t2.large VM.
* Ensure the following tools are installed and available in the PATH:
  * [podman](https://podman.io/getting-started/installation#linux-distributions)
  * [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
  * [minikube](https://minikube.sigs.k8s.io/docs/start/)

### Minikube installation:

It's convenient to just use the podman driver for minikube; it's lightweight and works great. We have some helper scripts that you will use to spawn a couple of minikube clusters to migrate between (let's call them src, dest) for crane, and we'll also set up some iptables network forwarding rules along with some in-cluster dns to ensure the clusters can route traffic to one another, and are able to resolve ingress. We also have scripts to help clean this up.

Here is a script to help you setup two minikube clusters and the appropriate iptable rules.

```bash
wget "https://raw.githubusercontent.com/konveyor/crane/main/hack/minikube-clusters-start.sh"
chmod +x minikube-clusters-start.sh
./minikube-clusters-start.sh
```
>**Note** - We also have had folks run through the scenarios on Mac machines, but fair warning, this path has significantly less usage.

>**Note** - Tackle requires a more demanding Minikube instance. Details about how to configure Minikube for the Tackle scenario can be found [here](/tackle/README.md#installing-minikube).

## Crane specific prereqs:

The crane scenarios utilize several additional tools. Ensure the binaries are
installed to your PATH so they are accessible:

* [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/)
* [jq](https://github.com/stedolan/jq)
* [yq](https://github.com/mikefarah/yq)
