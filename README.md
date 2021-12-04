
# OpenStackClient
** **

# OpenStack

OpenStack is an open-source platform that allows you to create private and public clouds. OpenStackClient (OSC) is the unified client for interacting with OpenStack-based clouds. It is a Python-based command line program that supports most OpenStack services, including the six core services: Compute (nova), Networking (neutron), Block Storage (cinder), Identity (keystone), Inventory (placement), Images (glance). It also provides a plugin architecture to allow users to extend commands or add support for non-core services. The unified OpenStack client is intended to replace a large number of project-specific clients.
See:
* [Openstackclient](https://docs.openstack.org/python-openstackclient/latest/)
* [OpenStack Services](https://www.openstack.org/software/project-navigator/openstack-components#openstack-services)

# Switch nova commands from using novaclient to using OpenStackSDK

## 1.   Vision and Goals Of The Project:

OpenStack is a set of components, both services and libraries. Most OpenStack componenets are managed by separate teams and therefore work a little differently from each other. This causes problems such as different versioning systems, or different command patterns leading to additional learning cost for OpenStack users.

Two projects aim to address this gap. Firstly, the unified OpenStack client, openstackclient or "OSC", aims to provide a single entrypoint to interact with all OpenStack services from the command line, masking differences in the underlying APIs and terminologies. This replaces the CLIs provides by the individual clients. The other project is the unified OpenStack SDK, openstacksdk. This provides a single way to interact with an OpenStack cloud programmatically via Python. This replaces the libraries provided by the individual clients. It aims to support all versions of the APIs provided by these services, past, present and future.

![alt text](https://github.com/DiweiZhu/OpenStackClient-CS6620NEU21Fall/blob/main/VisionGoals.png)

openstackclient is intended to replace the project-specific clients used in the past, such as novaclient and neutronclient, and offer a unified client tool to use OpenStack services. However, while projects such as the Networking service (neutron) have switched to using openstackclient as their primary CLI - either via integration in openstackclient core or an openstackclient plugin - other services have not yet provided full feature parity in openstackclient.

The focus of this project is on nova and cinder. High-level goals of this project include:

* Migrate openstackclient from project-specific libraries (for nova and cinder) to openstacksdk

* Identify CLI gaps (commands that are not supported yet) in OpenStackClient's support for the Block Storage service (cinder) and achieve feature-parity with ([python-cinderclient](https://opendev.org/openstack/python-cinderclient))

## 2. Users/Personas Of The Project:

OpenStackClient (aka OSC) is a command-line client for OpenStack that brings the command set for Compute, Identity, Image, Object Storage, and Block Storage APIs together in a single shell with a uniform command structure.

OpenStack is used by large enterprises, governments, and startups to build public and private clouds. Achieving feature-parity between project-specific clients and the unified CLI tool will benefit all users of OpenStack looking for a single easy-to-use client that supports most of OpenStack's services.

** **

## 3.   Scope and Features Of The Project:

* The first task is to migrate openstackclient from using project-specific libraries to using openstacksdk. This task needs to be done for both nova and cinder. For cinder, this will also resolve the issue of cinderclient dropping support for v2 of the API. This item is impacting users and is a priority for the community.
        * Commands for both nova and cinder can be found here: https://github.com/openstack/python-openstackclient/tree/master/doc/source/cli/data
        * The OpenStackClient repository: https://github.com/openstack/python-openstackclient
        * APIs for nova: https://docs.openstack.org/api-ref/compute/
        * APIS for cinder: https://docs.openstack.org/api-ref/block-storage/
	* These commands should be properly grouped and treated as groups
	* Commands using novaclient/cinderclient methods should be modified to use OpenStackSDK methods
	* Cinder commands in OpenStackSDK still supports v2.0 API, which solves the issue mentioned above

* There are a handful of commands and options that exist in the project-specific clients but not in OpenStackClient. There are a lot of commands already identified by the OpenStack team and have already been transitioned to openstacksdk. There are a few gaps left. The first of those remaining gaps is within cinderclient and openstackclient. Once these gaps have been closed, the Nova team can consider deprecating novaclient.
	* All nova commands already exist in OpenStackClient.
	* Must inspect these commands to check if their syntax is consistent, and if they work as intended.
	* For those that do not perform as desired, modify corresponding code, either using nova client methods or migrate to OpenStackSDK.
	* For those that are not using OpenStackSDK, have them use OpenStackSDK.

** **

## 4. Solution Concept

As mentioned above, in a high level, OpenStackClient should utilize methods from OpenStackSDK to provide users with consistent CLI commands for OpenStack services.

The design for that purpose is:
1. Add commands of services (nova, neutron, cinder, glance, keystone) into OpenStackClient, so that users have a CLI with a consistent style. For this, it doesn't matter if the OpenStackClient is using client-specific libraries or OpenStackSDK, because the behavior should be the same.
2. Migrate OpenstackClient from using client-specific libraries to using OpenStackSDK.

For our project, we need to do:
1. Migrate nova and cinder commands from using their specific client to using OpenStackSDK.
2. Find cinder commands that are supported in cinderclient but not supported in OpenStackClient, and add them into OpenStackClient.

For the first task, our methodology is:
1. Find out all commands of the two services in OpenStackClient, that are still using either novaclient or cinderclient methods.
2. For all of these commands, track them in the code base and find out which APIs their client-specific methods are ultimately calling.
3. Find corresponding methods in OpenStackSDK that are calling the same APIs.
4. If the corresponding methods exist in OpenStackSDK, change code of the commands to use the methods.
5. If they do not exist, send requests to the API directly in OpenStackClient.
6. Write unit tests and functional tests to make sure the modified methods act as expected.

For the second task, our methodology is similar:
1. Find out all commands that exist in cinderclient but not in OpenStackClient, by listing all commands and comparing.
2. For the commands we need to add into OpenStackClient, find out which APIs their client-specific methods are ultimately calling.
3. Find corresponding methods in OpenStackSDK that are calling the same APIs.
4. If the corresponding methods exist in OpenStackSDK, change code of the commands to use the methods.
5. If they do not exist, send requests to the API directly in OpenStackClient.
6. Write unit tests and functional tests to make sure the new added methods act as expected.

## 5. Acceptance criteria

Two major tasks are envisioned as an end goal for our project: OpenStack’s task to transition all project-specific clients to openstackclient has already had significant progress prior to starting our project. Some clients have already been replaced. Our aim is to migrate the client for Nova(Compute service) - novaclient to OpenStackSDK, and Cinder(Block Storage service) - cinderclient to OpenStackSDK. The second goal is to identify commands and options that exist in the project-specific clients but not openstackclient and then add these missing commands to openstackclient.

It may be a challenge anticipating how long our tasks may take given the stringent contributor standards, processes, and general unfamiliarity with the massive codebase. Based on our mentors' estimate, we may only be able to complete one of these tasks. The cinder is harder than the nova as our mentors have already done part of nova, we may set out to complete nova first, and treat cinder as a stretch goal.

## 6. Accomplishment

### Development Environment Setup

A devstack is setup for our development. DevStack is a series of extensible scripts used to quickly bring up a complete OpenStack environment based on the latest versions of everything from git master. It is used interactively as a development environment and as the basis for much of the OpenStack project’s functional testing. For each command we are working on, this devstack is uttilized to test the functionality of the command.

Ssh jump server is setup. Because of the fact that we need 5 development instance for our 5 members and the have only 2 floating IP addresses in our OpenStack project, we need to setup an ssh jump server so that everyone of us can ssh to the development instance of our own.

### Nova commands migrated

The commands we have migrated (12 in total):
- server backup create
- server image create
- server suspend
- server resume
- server pause
- server unpause
- server add volume
- server remove volume
- server add network
- server remove network
- server add port
- server remove port

The commands that are under review (9 in total):
- server list volume
- server lock
- compute service list
- compute service set
- compute service delete
- server start
- server stop
- server group show
- server add fixed ip

The commands that are in development (4 in total):
- server dump create
- server show
- server create
- server update volume

### Modification to OpenStackSDK

In some special cases OpenStackSDK might does not provide full support for a command or has some bug.
- Add OpenStackSDK method to support command server dump create.
- Fix bug in class definition of ServerDiagnostics
- Add \_max_microverion into ServerDiagnostics so that it calls new version of API by default.

## 7. What We Learned

### What is OpenStack?
We learned that OpenStack is a set of software components that provide common services for cloud infrastructure. It contains several components providing APIs to access key infrastructure resources. With OpenStack we can set up your own private cloud, using our very own bare-metal servers and it is entirely API driven, suited for a containerized world.

### OpenStack Philosophy, Guiding Principles, and Organisation

The Four Opens are a set of principles guidelines that were created by the OpenStack community as a way to guarantee that the users get all the benefits associated with open source software, including the ability to engage with the community and influence the future evolution of the software. Under these guidelines, the OpenStack community grew from tens of contributors to thousands, creating one of the top three most active open source projects in the world behind the Linux Kernel and Chromium. The Four Opens were instrumental not only in this success but also in building a thriving, global community.

OpenStack also has a set of guiding principles that are used to inform and shape decisions. These principles are not aspirational. Rather, they form the bedrock upon which the OpenStack community and software are built.

We learned about how OpenStack is structured as an organization that is governed by two separate governance bodies, formally defined in the OpenStack Foundation bylaws: the Board of Directors and the Technical Committee. It also comprises project teams and SIGs.

### What is OpenStackSDK?
We learned that OpenStack SDK is a client library for building applications to work with OpenStack clouds. The project aims to provide a consistent and complete set of interactions with OpenStack's many services, along with complete documentation, examples, and tools.

### IRC
We learned that IRC, or Internet Relay Chat, is often used as a real-time communication capability with open source projects. We got set up on IRC and joined relevant channels and explored useful commands to use IRC to connect with our mentors and the OpenStack community.
Main channels are:
* #openstack (for usage questions)
* #openstack-dev (for development questions)
* #openstack-infra (for questions about the project infrastructure)
* #openstack-upstream-institute (for beginners in the OpenStack land)

### Using Gerrit with Git
Gerrit is the code review system used by the OpenStack project. We learned about the development and code review workflows for projects hosted on the OpenDev infrastructure. OpenDev has a sandbox repository for learning and testing purposes: https://opendev.org/opendev/sandbox. This was a great repository to begin our OpenDev learning. We experimented with the workflow and tried different options to learn what they do.

### DevStack
DevStack is a series of extensible scripts used to quickly bring up a complete OpenStack environment based on the latest versions of everything from git master. It is used interactively as a development environment and as the basis for much of the OpenStack project’s functional testing. We set up our own DevStack instance for the team on one of MOC's VMs and exposed it to our individual dev instances to test our code against DevStack. We learned about clouds.yaml configuration to have all our dev instances talk to DevStack.

### Agile development and Taiga
We learned about agility in software development, SCRUM, and its benefits. Working in sprints and regularly sharing our progress with the class has helped us practice agile development and understand its value. Using Taiga helped us stay accountable and track our progress against our goals. Planning sprints, estimating story points, and reviewing burndown charts were useful exercises to get a glimpse of how the software industry works today.

### Tox
We learned about standardized testing in Python using tox. tox aims to automate and standardize testing in Python. It is part of a larger vision of easing the packaging, testing, and release process of Python software. tox is a generic virtualenv management and test command-line tool you can use for:

* Checking that your package installs correctly with different Python versions and interpreters
* Running your tests in each of the environments, configuring your test tool of choice
* Acting as a frontend to Continuous Integration servers, greatly reducing boilerplate and merging CI and shell-based testing.

### API Microversioning
We learned about the API Microversions framework in OpenStack used to allow changes to the API while preserving backward compatibility. The basic idea is that a user has to explicitly ask for their request to be treated with a particular version of the API. So breaking changes can be added to the API without breaking users who don’t specifically ask for it. This is done with an HTTP header OpenStack-API-Version which is a monotonically increasing semantic version number starting from 3.0.

### Zuul and Project Gating
Zuul is a program that drives continuous integration, delivery, and deployment systems with a focus on project gating and interrelated projects.
* Zuul’s configuration is organized around the concept of a pipeline. In Zuul, a pipeline encompasses a workflow process that can be applied to one or more projects.
* Pipelines have associated triggers which are descriptions of events that should cause something to be enqueued into a pipeline.
* Once all of the jobs for an item in a pipeline have been run, the pipeline’s reporters are responsible for reporting the results of all of the jobs.

The process of project gating attempts to prevent changes that introduce regressions from being merged. This keeps the mainline of development open and working for all developers, and only when a change is confirmed to work without disruption is it merged. Zuul can help automate this process, with a particular emphasis on ensuring large numbers of changes are tested correctly.
> Why Zuul replaced Jenkins: https://superuser.openstack.org/articles/introducing-zuul-for-improved-ci-cd/

### cliff
Command Line Interface Formulation Framework (cliff). cliff is a framework for building command-line programs. It uses plugins to define sub-commands, output formatters, and other extensions. We learned about its syntax and how it is used in OpenStackClient. We also explored OpenStackClient hierarchy with cliff, osc-lib, and OpenStackClient.
* cliff: a framework for command-line programs that uses entry points to manage subcommands
* osc-lib: general part of OSC that connects to the cloud and does most of the authentication
* OSC: provides information of services and commands and does a small part of the authentication

### Unit and functional testing
We learned to write useful unit tests to instill confidence that our implementation doesn't just work but that it always works. OpenStackClient also maintains a set of functional tests that are optimally designed to be run against OpenStack’s gate. Our implementation also required that we make changes to functional tests of OpenStackClient to ensure our changes work as intended with specified functional requirements of the project. Commands are tested by feeding them input and examining the output, and internal program structure is rarely considered. Functional testing usually describes what the system does.

### Opensource development
Open-source software is software with source code that anyone can inspect, modify, and enhance. We learned that contributing to open source can be a rewarding way to learn, teach, and build experience in just about any skill you can imagine. We were able to take a code change from start to end to get it merged into production of an open-source project while maintaining coding standards, addressing code review feedback, following style guidelines, and writing and passing useful tests. This has been a challenging and rewarding experience.


## 8. Main Obstacles

## 9. Future Work

During the entirity of our work for OpenStackClient we were faced with some challenging situations and their resolution depended on implementing new changes in the OpenStackSDK before we could migrate some commands. As discussed in the obstacles <>
## 10. Release Notes: [need refine]

Taiga Board: https://tree.taiga.io/project/amanbatra-cs6620-fall21-openstackclient/timeline

Sprint | Tasks
:------: | ------
1 | Setup the OpenStackClient environment on the Cloud VM: <br><ul><li>Create accounts for IRC and Gerrit</li><li>Set up a Linux-based development environment</li><li>Set up an all-in-one OpenStack deployment that you can use for testing</li></ul></li></ul>
2 | The next step is to go through the source code and get familiar with the functionality of Nova client service. This is crucial in understanding the gaps to figure out how to migrate the remaining commands that still use native Nova API’s and libraries and migrate them to OpenStackSDK. To completely understand the OpenStackClient functionality we need to execute the basic set of commands like create and modify Cloud environments.
3 |We shall focus on bringing the commands that are missing in OpenStackSDK for Nova client. This is done by adding the proxy classes/methods in the OpenStackSDK for Nova. After this, it is critical to run a proper testing suite to make sure there is no impact on the system.
4 |The next task is to plug the gaps in commands that are already in the OpenStackSDK. We do this by debugging why the command is not working as expected and adding/fixing the code.

### Mentors:
* Artem Goncharov <artem.goncharov@gmail.com>
* Stephen Finucane <stephenfin@redhat.com>
* Kendall Nelson <kennelson11@gmail.com>

