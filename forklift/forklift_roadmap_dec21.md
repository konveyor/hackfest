# Topics

## VMware SSL Fingerprint

The goal of this field is to improve security. When a user provides an IP address or hostname for the vCenter API endpoint, the provider controller needs to verify the server certificate, in order to avoid man-in-the-middle (MITM) attacks. This is important because we send the VMware username and password to that endpoint. So, if a MITM attack is undergoing, the user may disclose credentials for a highly privileged account.

There are basically two options to verify the server certificate: 1) provide the CA certificate chain for the server certificate to the provider controller, 2) ask the user to provide the fingerprint for the provider controller to compare with the one exposed by the endpoint. In any case, the user needs to provide something to allow the provider controller to trust the vCenter endpoint. We considered that getting the fingerprint was not very difficult and would require less operations.

That being said, I understand that users find it painful to switch to the command line to get the fingerprint and then paste it in the UI. Jeff proposed an option where the UI would get the server certificate and then would display some attributes of the certificate (subject, issuer, validity and fingerprint), alongside with a checkbox that the user needs to check to confirm that the certificate is valid. The experience would be similar to checking the box to accept General Conditions of Usage.

The UI would still only set the fingerprint when creating the provider secret, so the only change would be on the UI side. A quick Google search gave me this old example: https://stackoverflow.com/questions/13596315/how-to-get-ssl-certificate-information-using-node-js.

### Verify Buttons
Have verify buttons for credentials.
https://bugzilla.redhat.com/show_bug.cgi?id=1958834 
How can we have verify buttons when we add a new credential, before storing them.
Feedback captured from an IBM team installing MTV for the first time was that the fingerprint field was confusing: 
Couldn’t find it in vCenter
Requesting it from a different workstation was not seen as the best method (specially since they have Windows desktops)
Not appearing in import wizard

## VDDK init image

To perform the disk transfer for VMware VMs, Matthew has implemented a CDI datasource that leverages nbdkit and its VDDK plugin. For licensing reasons, Red Hat cannot ship the VDDK library. This is why we ask the users to create and publish an image with the VDDK library, then to set the vddk-init-image in the OpenShift Virtualization configuration. This is not strictly an MTV requirement, but migrations will fail without it.

The documented steps [1] seem to be too difficult to follow or not clear enough. We can make the documentation more explicit, add examples. But my understanding is that it might not be enough, because users don't read the documentation. So, we can explore a few technical options.

The first one is to check the presence of the vddk-init-image setting in the OpenShift Virtualization configuration and add a condition on the Provider CR to reflect that. This would allow the UI to display a warning icon on the provider, in order to raise users' awareness. In MTV, we support many destination providers, so the fact that it's set on the cluster where MTV is installed doesn't mean it's installed on all the other target OpenShift clusters. This seems like a valid check to add to the Provider CR validation.

But that doesn't solve the image creation. We could add an action on the provider in the UI to trigger the build of the image, but that would require that:
The user provides an unauthenticated URL where the VDDK tar.gz is stored. This URL must be accessible from the target OpenShift cluster. This means that the user needs to 1) download the tar.gz (and not the zip) from VMware customer portal, 2) upload the tar.gz to an HTTP server. I anticipate that setting up the HTTP server will be too much for most users.
We implement a helper to trigger the build / push of the image to a given registry. The build / push needs to happen in a pod on the target OpenShift cluster, which means additional permissions for MTV. Then, to push to a registry, we can consider the internal registry, but it may not be deployed. In that case, the user needs to provide the credentials to push to another registry, that will be stored in a Secret in the target OpenShift cluster.
I am not sure that the benefit / risk balance is in favor of such a complex solution.
If you have simpler ideas, I'm listening.

[1] MTV 2.1 User Guide - Creating a VDDK image.

## VMs with incompatible names

Kubernetes only allows names in the RFC 1123 DNS Subdomain format for the objects [2]. This also applies to VM names.
On the VMware side, almost any character is allowed in a VM name, so customers have implemented naming conventions that make VMs incompatible with Kubernetes.
To unblock that, we currently recommend renaming the VMs on the VMware side with valid Kubernetes names. But that is very intrusive and will likely be rejected.
Another solution would be to rename the VMs on the fly and there are options:
Changing the VM name in the MTV Plan wizard - This would allow the user to set the new name VM by VM when creating a plan. @Vince Conzola, would that fit in the UI?
Ask the user to provide a regex to replace invalid characters - The plan controller would use the regex and generate new names with a one time interaction. This is better for changes at scale, but it requires strong regex knowledge from the users. To limit the risk of generating unexpected names, the UI and the plan controller can preprocess all the names and show what they would become.
Provide a regex builder from the identified naming issues in the inventory. For example, the inventory has identified VMs with underscores in their names, then the user can select the hyphen character to replace the underscores.
[2] https://kubernetes.io/docs/concepts/overview/working-with-objects/names/

## Network Configuration Transfer

How to generate the same networks in OpenShift that were available in source?

## OPAL

https://issues.redhat.com/browse/MTV-17 
Video: https://drive.google.com/drive/folders/1awd2DD-cIyu5d1BK7lhKbD200SakRtX7 

Actions for Product Manager
Define user flow related to how data is gathered for VMs and resources (network and storage)
Paths to decide and create migration plans.


## OpenStack Advanced Migration Features

When looking at OpenStack and OpenShift, one can see many similarities. The main one is that they split the control and data planes with core services (mysql, oslo vs etcd, kubeapi), infrastructure services (metrics, logging….) and workload services (nova, neutron… vs. pod scheduler, multus…). This approach is what allows them to scale massively and offer a broad ecosystem of services.

To the point where they interconnect. OpenStack has Magnum to manage container stacks on OpenStack, while OpenShift has OpenShift Virtualization to run virtual machines on OpenShift. And OpenShift on bare metal is a trending topic, while OpenStack runs its control plane as containers. The lines are blurring.

The NFV ecosystem is still heavily focused on virtual machines, while the containers are emerging. With OpenShift Virtualization, the Telco customers can make virtual machines and containers coexist on the same platform, until they have fully certified stacks based only on containers. But if we don't consider the whole workload migration, but only virtual machines, we will have hard times to convince these OpenStack champions to move to OpenShift.

This is why we propose to look at this migration holistically, addressing compute, network and storage at once. Some feature mappings are more obvious than others, but here are some examples:

* Virtual machine - This is the most obvious one and the core of the migration topic. It will be a virtual machine in OpenShift with a simple mapping of its metadata. There's no need to change the operating system and cloud-init is supported.

* Virtual machine image - OpenStack allows storing instance images in Glance, either to import a fully configured VM or to use as templates for new instances. OpenShift Virtualization implements a similar mechanism through container images stored in a container registry.

* Volume - A volume is generally assigned to a virtual machine as a new block device. The migration would turn it into a Persistent Volume that could be attached to a virtual machine or to a pod.

* Storage swinging - An option to migrate large amounts of volume would be to connect OpenShift to the same backend as OpenStack Cinder, if a CSI driver exists. The volumes could then be mapped to Persistent Volumes and reused without moving the data.

* External network - In OpenStack, an external network is used for direct layer 2 connectivity in the virtual machines, avoiding the SDN overlay, or to allow virtual machines to connect to non overlay networks through a router and maybe a floating IP. The notion of router doesn't exist, but multus networks allow the layer 2 connectivity or masquerading for egress traffic, while the service/route can allow ingress traffic.

* Load balancer - In OpenShift, the Service object allows various strategies, among which load balancing. This allows to expose a single entrypoint for many virtual machines with balancing rules. And if the service is HTTPS based, the route can also provide TLS termination, with re-encryption capability.

* Orchestration Stack - To abstract a complex workload deployment and handle horizontal scalability, OpenStack has the Heat service that allows describing the application components, with variables that can be set at runtime. OpenShift proposes a similar mechanism through either a Template or a Helm Chart. The OpenStack vocabulary is wide, so we could focus on the resources supported by Red Hat OpenStack Platform. The gaps with OpenShift could also be prioritized.

* Project - Both OpenStack and OpenShift use the concept of project as the core concept of their multi-tenancy. And they both support assigning roles to users or groups, and to manage resource quota. So, keeping the migrated resources in the same project structure seems natural.

* From a technical point of view, we would use the OpenStack stable APIs to discover the resources, like os-migrate does. However, for a better integration in MTV and with the OpenShift ecosystem, the development would be done in Golang, probably with gophercloud.

A strong emphasis should also be put on the validation rules and user experience to ensure that the users are aware of the changes 

### OpenStack non-RH providers

About using common APIs to extract data … but which common APIs? OSP 13 and 16 and their underlying versions? All the in betweens? Which ones align to Windriver, Huawei, and friends? These details will matter and may result in you not having coverage where you think you do when speaking to third party solutions.


## Migrating GPU enabled VMs

 From Peter Lauterbach:
“Thinking more about the AI.ML use case that is virtualized on vSphere.  Customers already have GPU enabled VMs that they are using in AI/ML pipelines for things like training models.

Is it possible to detect this configuration, and map it into a similarly configured VM on the OCP side? Most likely they would be using the same GPU hardware and driver on both sides, but possibly new models on the OCP side.

I'm trying to elevate my thinking from just the VM mechanics perspective, to "how can we make it easier for customers to move entire applications (that are based on VMs), into OCP.”

## Ansible Hooks Image Builder
Use cases draft
Modify external services related to the VM 
Load Balancer
DNS
Monitoring
Modify the VM itself 
Remove unneeded software (i.e. agents, virt-tools, driver related apps)
Apply patches or upgrades (i.e. to adapt to target environment)
Add software (i.e. agents, virt-tools, extra drivers)
Change IP addressing (i.e. not possible to have routed VLANs)
Change VM Attributes (name, cores, memory) ? [Note: Not sure if this is for hooks]
 
Run tests
Custom pre-migration checks
Automated conformance post-migration testing 

Under consideration: Requiring external Tower to execute hooks
Note: In some cases like wanting to keep IPs assigned to MAC addresses, the pre-migration runs will be required.  It may be too much to request the user to do.

### Existing Ansible roles and playbooks:
https://github.com/fdupont-redhat/ims-ansible-playbooks
https://github.com/fdupont-redhat/ims.premigration-rhel
https://github.com/fdupont-redhat/ims.premigration-windows
https://github.com/fdupont-redhat/ims.postmigration-convert2rhel

The hooks as implemented in MTC proposes two experiences:
Run a custom container image provided by the user. This is the most versatile approach, as the user has complete freedom with regards to the technology in the container image and how it implements the hook actions.
Execute an Ansible playbook provided by the user as a single file. Behind the scenes, it will run a pod based on a Red Hat-customized Ansible Runner image. This reduces the complexity of creating and publishing a container image by proposing an opinionated automation solution, Ansible.

Our experience with Ansible in the field has shown that Ansible playbooks can be more complex than a single file. It is very common that the playbook will call custom roles, filters, plugins and even modules. These assets are frequently managed in a source code management (SCM) system and can be published in Ansible Galaxy, allowing for a consistent company-wide automation framework, reused across all entities. This is why we want to expand the hooks user experience to provide an opinionated integration with Git and Ansible Galaxy for these customers.

One important consideration is reproducibility. When users run a migration plan, they expect that the same hooks are applied to all the virtual machines in the plan. If the assets are retrieved at run time and some virtual machine migrations are postponed by the throttling mechanism, it may happen that the hook behaves differently because some asset has changed between the plan start and a given virtual machine migration. The solution must provide a mechanism to ensure that all the virtual machines in a plan run the same hooks.

### Possible Implementations
Use the current MTC implementation: It doesn’t seem to cover all the needs of MTV
Implement an “Ansible Runner” for MTV: Could mean an important investment of time. May affect delivery dates.
It may become a joint project with MTC by extending the capabilities that they have. It can even become a module for anything requiring automation within OpenShift.
Request to have an external Ansible Tower: Putting a lot of load on Users.
Providing an initial playbook to prepare tower and example playbooks would reduce the effort to have it running
Could also mean a Tower upsell.
Embed AWX/Tower: Difficult from the maintenance point of view. It will increase the footprint of the tool a lot
Already tried by CloudForms and ended up in reimplementation as Ansible Runner.

### Feedback received so far
The current implementation of hooks in MTC does not fit completely on what MTV needs. There is an increased complexity in managing a different platform, and having to handle different credentials (VMs and external services). We will need to add more capabilities to that feature
The embedded implementation, a la "Ansible Runner" is the preferred choice as the tool will include all that is required to perform a migration, and will suit customers with a low level of automation (or automating with other tools)
Using an external Ansible Tower would help for large migrations and is a good feature to be considered down the path but seems to be an overkill to consider it the only option.
Embedded Ansible Tower/AWX was discarded as unmaintainable and difficult to handle (references to CloudForms initial implementation).

The suggested path, with current feedback, is to extend the capabilities of the MTC implementation to cover the needs of VMs. Having a common component for MTV and MTC to be considered.

### Considerations
External Tower is too complex for mid size migrations, and embedding it will be a maintenance issue.
The embedded use case will be the way to go. Ansible Runner could be considered an example.
Having a user provided container image for the user to run, does not seem to apply.
We will need to consider that the automation will be: 
Making Changes in the VM ot be migrated 
Making changes in the external infra (LB, DNS, Monitoring)
The number of integrations to cover is much larger than with containers
The complexity of playbooks will be greater than with containers and will be very likely hosted in a git repository
We still want to make the user be able to paste variables (and credentials) for each run to customize them.

### Solution
After reviewing the considerations we worked on this set of Mockups to cover the needs and situation found: Mockups for MTV 2.1 latest 

For a given playbook, the same set of playbook/dependencies must be run on each VM in a given plan for a given run.
