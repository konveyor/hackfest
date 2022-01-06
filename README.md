# hackfest

## Prerequisites

We recommend bringing a RHEL-family machine (Fedora 35, Centos 8, RHEL 8) with podman installed. A VM should work fine. You'll also need minikube available. It's convenient to just use the podman driver for minikube; it's lightweight and works great. We have some helper scripts that you will use to spawn a couple of minikube clusters to migrate between (let's call them src, dest), and we'll also set up some iptables network forwarding rules along with some in-cluster dns to ensure the clusters can route traffic to one another, and are able to resolve ingress. We also have scripts to help clean this up.

### VM size:
we recommend a minimum of 2vCPUs and 8GB of RAM.  If using Amazon EC2 instance, we recommend a t2.large VM.

### Minikube installation:
As per the above description, here is a script to help you setup two minikube clusters and the appropriate iptable rules.
```bash
curl -s "https://raw.githubusercontent.com/konveyor/crane/main/hack/minikube-clusters-start.sh" | bash
```

NOTE: We also have had folks run through the scenarios on Mac machines, but fair warning, this path has significantly less usage.

Have jq installed on your system and available inside of your path.

Have yq installed on your system and available inside of your path.
