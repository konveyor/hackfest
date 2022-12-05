# Move2Kube Hackathon

## Prerequisites for the workshop

- Install [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) if you are using a Windows machine.

- To build container images you will need a container runtime like [Docker](https://www.docker.com/get-started) or [Podman](https://podman.io/getting-started/installation).

- To push and pull the container images to an image registry, you will need an account in an image registry like [quay.io](https://quay.io), [Docker Hub](https://hub.docker.com/), etc.

- (Optional) To deploy the output of Move2Kube, you need a Kubernetes cluster. Since most people don't have access to a cluster we recommend [Minikube](https://minikube.sigs.k8s.io/docs/start/).

## Topics

* [Move2Kube - CLI and UI installation.](./installation)
* [Migrate Language Platforms sample apps to Minikube/Kubernetes.](./exercise-1-language-platforms/)
* [Migrate a CLoud Foundry enterprise app to run on Minikube/Kubernetes.](./exercise-2-cf-enterprise-app/)
* [Move2Kube - Advanced features. Customizing the migration process using transformers.](./exercise-3-customizations/)

## Links

* [Tutorials](https://move2kube.konveyor.io/tutorials)
* [Documentation](https://move2kube.konveyor.io/documentation)
* [Move2Kube Github](https://github.com/konveyor/move2kube): File bugs and feature requests here, using the dedicated issue templates.
* [Slack](https://kubernetes.slack.com/archives/CR85S82A2)