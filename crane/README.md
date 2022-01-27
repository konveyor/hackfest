# Crane Hackathon

## Event #1 (December)

### Agenda - day 1 (Monday, Jan 31st)
* 9:00am Getting ready for today and tomorrow
* 9:10am Quick update on Crane
* 9:45-11:45am  Getting started together
* 1pm-5pm Running through scenarios, answering questions, filing BZs, etc...

### Agenda - day 2 (Tuesday, Feb 1st)
* 9:00am Monday recap
* 9:15am Discussion with Engineering, early feedback, etc.
* 10am More hands-on on scenarios
* 3pm Retro on what we learned + Capture final feedback, questions, improvements, etc. 


## Links

* [Agenda and Intro slides](https://docs.google.com/presentation/d/1pOZOTv_8p9lIKbp8-R5291OfyMGkdZDTT9AYxSVgJTU/edit?usp=sharing)
* [Upstream Documentation](https://crane-docs.konveyor.io)
* [Documentation Github](https://github.com/konveyor/crane-documentation): File issues with docs here
* [Crane Github](https://github.com/konveyor/crane): File bugs or RFEs here,
using the dedicated issue templates.
* [Crane Runner Github](https://github.com/konveyor/crane-runner)

## Prerequisites

See [top level prereqs](../README.md#prerequisites) to prepare your infrastructure needs.

The crane scenarios require some [additional tooling](../README.md#crane-specific-prereqs)

## Some background

Crane itself is a cli tool that's designed to help users with application
portability. It is designed with the unix philosophy in mind, meaning that you
could consider it a swiss-army knife of tightly focused and sharp tools that
are intended to work in conjunction with one another to achieve more sophisticated
things. It can be used directly on your working machine, and uses a standard
kube config to communicate with your clusters, accepting a kube context specification
to understand the coordinates to the cluster you're trying to operate upon.

Because of its "pipelined" nature, it pairs very nicely with Tekton (OpenShift pipelines).
We are working on a product that we expect to be shipped sometime in the Spring
of 2022 that is installable via operator, augments the OpenShift console directly,
and exposes a nice UI for driving migrations. Under the covers, it will depend
upon OpenShift Pipelines, and will use crane to migrate applications around.
Crane will execute its tasks within a pipeline, embedded inside of an image.
That image and the associated Tekton assets that help you run pipelines to
do useful things is called [crane-runner](https://github.com/konveyor/crane-runner).

During this hackfest, we'll use crane-runner and these Tekton tasks to run
crane within your cluster, and execute migrations.

## Additional Resources

* Tekton has excellent documentation. Feel free to check out
    [Tekton's Overview](https://tekton.dev/docs/overview/). Crane Runner --
    `crane` shoved into a container image -- primarily uses Tekton's
    ClusterTasks, TaskRuns, and PipelineRuns.
* [Kustomize](https://tekton.dev/docs/overview/) came out of the
    ["Declarative Application Management" Whitepaper](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/declarative-application-management.md)
    by Brian Grant. It's primary benefits are 1) built-in directly to `kubectl`
    command line tool (the `--kustomize` flag on create/apply) 2) allows for
    kustomization of application without modifying the base manifests and 3) no
    special templating layer. We envision Crane Runner leveraging Kustomize as a
    mechanism for _adding back_ cluster specific details on "destination"
    clusters.
* [Argo CD](https://argo-cd.readthedocs.io/en/stable/)

## Scenarios

We have added several examples to crane-runner that gradually ratchet up the
complexity of the overall scenario, and demonstrate the value of crane with
one possible usage of the tool. Again, because it is a toolbox of utilities,
it's extremely flexible in what it can do.

Our scenario instructions can be found in our [crane-runner examples](https://github.com/konveyor/crane-runner/tree/main/examples).

## Support and filing bugs

Please join the #konveyor channel in [Kubernetes slack](https://k8s.slack.com/) for support and
discussion. We have an issue template in the crane-runner repo that can be
used specifically for the hackathon as well as an RFE template for suggested enhancements.
If your issue is not specific to crane-runner and the tasks, but the crane tool
itself, please consider filing it in the [crane repo](https://github.com/konveyor/crane).

Thank you for attending!
