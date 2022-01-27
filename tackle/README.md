# Tackle Hackathon

## Links

- [Upstream Documentation](https://tackle-docs.konveyor.io/)
- [Documentation Github repository](https://github.com/konveyor/tackle-documentation): File issues with docs here
- [Tackle Github repository](https://github.com/konveyor/tackle): File bugs or RFEs here,
using the dedicated issue templates.
- [Tackle Workshop slides](https://docs.google.com/presentation/d/1zBstluASgDOs8Ju2vjtT5z-Ho2nIniFHXSDJk9DmfhY)


## Prerequisites

In no other Kubernetes cluster is available for the user, the first thing to do is to install Minikube, which provides a certified Kubernetes platform with most of the vanilla features. This allows to simulate a real Kubernetes cluster with a fairly high level of feature parity. Nevertheless, this is just the minimum recommended environment, and any other Kubernetes environment with OLM and an Ingress Controller properly configured would be enough.


>**Note** - The steps described below are executed on a Fedora 35 workstation, but will likely work on any recent Linux distribution. The only prerequisites are to enable virtualization extensions in the BIOS/EFI of the machine, to install libvirt and to add our user to the libvirt group.*


### Installing Minikube

The installation consists in downloading the `minikube` binary into `${HOME}/.local/bin`, so that we don't need to modify the PATH variable. Other options are described in Minikube documentation.

```shell
curl -sL -o ${HOME}/.local/bin/minikube \
    https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

The binary doesn't have the execution permission, so we add it.

```shell
chmod u+x ~/.local/bin/minikube
```

By default, Minikube uses the `kvm` driver with 6,000 MB of memory. This is not enough to run all the services, so we need to increase the allocated memory. In our experience, 10 GB of memory is fine.

```shell
minikube config set memory 10240
```

Then, the following command will download a qcow2 image of Minikube and start a virtual machine from it. It will then wait for the Kubernetes API to be ready.

```shell
minikube start
```

We need to enable the `dashboard`, `ingress` and `olm` addons. The `dashboard` addon installs the dashboard service that exposes the Kubernetes objects in a user interface. The `ingress` addon allows us to create Ingress CRs to expose the Tackle UI and Tackle Hub API. The `olm` addon allows us to use an operator to deploy Tackle.

```shell
minikube addons enable dashboard
minikube addons enable ingress
minikube addons enable olm
```

The following command gives us the IP address assigned to the virtual machine created by Minikube. It will be used later, when interacting with the Tackle Hub API.

```shell
$ minikube ip
192.168.39.23
```

### Configuring kubectl

Minikube allows us to use the kubectl command with `minikube kubectl`. To make the experience more Kubernetes-like, we can set a shell alias to simply use `kubectl`. The following example shows how to do it for Bash on Fedora 35.

```shell
mkdir -p ~/.bashrc.d
```

```shell
cat << EOF > ~/.bashrc.d/minikube
alias kubectl="minikube kubectl --"
EOF
```

```shell
source ~/.bashrc
```

### Accessing the Kubernetes dashboard

We may need to access the dashboard, either simply to see what's happening under the hood, or to troubleshoot an issue. We have already enabled the `dashboard` addon in a previous command.

We can use the `kubectl proxy` command to enable that. The following command sets up the proxy to listen on any network interface (useful for remote access), on the `18080/tcp` port (easy to remember), with requests filtering disabled (less secure, but necessary).

```shell
kubectl proxy \
    --address=0.0.0.0 --port 18080 \
    --disable-filter=true
```

We can now access the dashboard through the proxy.
In the following URL, replace the IP address with your workstation IP address.

`http://192.168.0.1:18080/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/`

## Installing Tackle

Please follow [the Tackle installation instructions available in the official documentation](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#installing-pathfinder_tackle) and [file a bug](https://github.com/konveyor/tackle-documentation/issues) in case you find inconsistencies or run into problems.

>**Note** - Please take into account that, depending on the resources allocated for your Kubernetes cluster and namespace, the tackle instance might take several minutes to have all its services up and running.

## Tackle Overview

Tackle is a toolkit aimed at helping organizations safely migrate and modernize their application portfolio to leverage Kubernetes. For the moment, it provides application portfolio management and assessment capabilities, although other features such as application analysis are planned for the upcoming future.

A complete overview of the services or modules currently available on the Tackle web console can be seen in the [Tackle web console services from the official documentation](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#console-services_tackle).

## Workshop

Once Tackle is up and running in you Kubernetes cluster, [login](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#accessing-console_tackle) to start working on the practical exercise to get familiar with the tool. **Links on the description of each step will lead to the section in the official documentation that explains how to perform the described operation.** If something is not clear enough, missing or simply wrong, **please [open an issue in the documentation repository](https://github.com/konveyor/tackle-documentation/issues)**. This would be of great help to make sure our docs are useful for users wishing to start using Tackle.

### Setting up the controls

These first steps will be focused in modeling the organization we will be working with inside the Tackle tool.

First of all, we will [create a new job function](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#creating-job-function_tackle): CEO & Owner

![Create job function](images/01_Create-job-function.png?raw=true "Create job function")

Let's [create a stakeholder](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#creating-stakeholder_tackle) now, and associate the recently created job function to it:

- email: hscorpio@globex.com
- Display name: Hank Scorpio
- Job Function: CEO & Owner

![Create stakeholder](images/02_Create-stakeholder-scorpio.png?raw=true "Create stakeholder")

Now [create a stakeholder group](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#creating-stakeholder-group_tackle) called *Senior Management* and add Hank Scorpio to it:

![Create stakeholder group](images/03_Create-stakeholder-group.png?raw=true "Create stakeholder group")

We will need to create two additional stakeholder groups: *IT Management* and *Retail Management*.

Once we have all the stakeholder groups available, let's create another stakeholder:

- email: hsimpson@globex.com
- Display name: Homer J. Simpson
- Job Function: Program Manager
- Group(s): IT Management, Retail Management

![Create stakeholder](images/04_Create-stakeholder-simpson.png?raw=true "Create stakeholder")

Finally, let's [create a business service](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#creating-business-service_tackle) named *Retail* under which all our applications for this workshop will lie:

![Create business service](images/04_Create-stakeholder-simpson.png?raw=true "Create business service")

### Managing the application portfolio

The first step to start working with applications will be to [import a list of applications from CSV](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#importing-applications_tackle) using the following sample file:

- [applications.csv](files/applications.csv?raw=true)

![Import applications](images/06_Import-applications.png?raw=true "Import applications")

After the import we should have the following list of applications available in the *Application Inventory* screen:

![Application list](images/07_Application-list.png?raw=true "Application list")

Clicking on each application will expand if, displaying information related to it such as the tags that were included in the file:

![Application details](images/08_Application-details.png?raw=true "Application details")

Let's create an application from scratch to complete the list of retails applications from the Globex corporation:

- Name: Discounts
- Business service: Retail
- Tags: Python, RHEL 8, MongoDB

![Create application](images/09_Create-application.png?raw=true "Create application")

Tackle uses an extensible tagging model to allow classifying applications in multiple dimensions. The tool comes with several tags and tag types out of the box, but in case these not cover our needs, it is fairly simple to add new ones. Let's begin by [creating a new tag type](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#creating-tag-type_tackle) named *Persistence*:

![Create tag type](images/10_Create-tag-type.png?raw=true "Create tag type")

Now, let's [create a couple of tags](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#creating-tag_tackle) for that tag type with some persistence frameworks such as *Hibernate* and *JPA*:

![Create tag](images/11_Create-tag.png?raw=true "Create tag")

Now that we have the new tags, let's [update the tags assigned to an application](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#updating-tags-of-application_tackle) for example by adding the *Hibernate* tag to the Inventory application:

![Assign tag](images/12_Assign-tag.png?raw=true "Assign tag")

Tags, among other fields, can be used to filter the application list displayed on the *Application Inventory* screen. Let's filter our applications and see which ones are made in Java:

![Filter by tag](images/13_Filter-by-tag.png?raw=true "Filter by tag")

### Assessing an application

Once the portfolio has been organized, we can start working on the assessment of our applications. Let's [start an assessment](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#starting-assessment_tackle) for the *Customers* application, adding Homer as the involved stakeholder and IT Management and Retail Management as the involved stakeholder groups:

![Select stakeholders](images/14_Select-stakeholders.png?raw=true "Select stakeholders")

Dependencies for an application can be managed during the assessment or in the *Application Inventory* view. In this case, we will do it on the assessment by clicking on the *Manage Dependencies* button and adding a northbound dependency to the *Gateway* application:

![Manage dependencies](images/15_Manage-dependencies.png?raw=true "Manage dependencies")


Now we can proceed with the questionnaire. Pathfinder will find risks based on the answers provided for each question:

![Answer questionnaire](images/16_Answer-questionnaire.png?raw=true "Answer questionnaire")

After we are done, the application will appear with its assessment completed in the application inventory.

![Assessment completed](images/17_Assessment-complete.png?raw=true "Assessment completed")

Next step should be [reviewing the assessment](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#reviewing-assessment_tackle) to understand the risks that were found and decide the most suitable action based on the results:

![Review screen](images/18_Review-overview.png?raw=true "Review screen")

Risks can be found at the bottom of the *Review* screen and ordered by their risk:

![Identified risks](images/19_Identified-risks.png?raw=true "Identified risks")

The top section of this screen allows to decide a proposed action and provide an estimate for the effort required to migrate the application along with its business criticality and work priority:

![Proposed actions](images/20_Proposed-action.png?raw=true "Proposed actions")

Assessing and reviewing applications one per one might not be practical when dealing with large portfolios, so Tackle allows [copying and applying assessments and reviews](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#copying-assessments-and-reviews_tackle) based on filtering criterias:

![Copy assessment](images/21_Copy-assessment.png?raw=true "Copy assessment")

Select all applications available in the inventory and click on *Copy* to get your assessment and review data carried over the whole portfolio:

![Select target applications](images/22_Select-target-applications.png?raw=true "Select target applications")

This is especially useful when dealing with different application types in a given portfolio, so for example these types could be modeled using a given tag, perform the assessment to then filter the application list to apply that same assessment to applications belonging of that same type.

![Copied assessments](images/23_Assessment-copied.png?raw=true "Copied assessments")

### Obtaining aggregated data in reports

[The *Reports* screen provides a way to get aggregated assessment and review data](https://tackle-docs.konveyor.io/documentation/doc-installing-and-using-tackle/master/index.html#about-reports_tackle). This is helpful to get an overview of the current situation when dealing with a large portfolio. The screen is divided in sections that focus on different aspects of the aggregated data. At the top of the page we have the *Current landscape* and *Adoption candidate distribution* section that offer an aggregated view of the assessment data and details about the review respectively:

![Reports screen](images/24_Reports-screen.png?raw=true "Reports screen")

The *Adoption candidate distribution* has a graph view that represents applications in a two-dimensional plane based on their Business criticality and Confidence:

![Adoption candidate distribution](images/25_Adoption-candidate-distribution.png?raw=true "Adoption candidate distribution")

The *Reports* screen also provides a suggested adoption plan for the assessed applications based on effort, priority and dependencies:

![Adoption plan](images/26_Adoption-plan.png?raw=true "Adoption plan")

Finally, at the bottom you can find a list of the severe risks identified across the application portfolio:

![Identified risks](images/27_Identified-risks.png?raw=true "Identified risks")
