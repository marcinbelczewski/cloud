# Cloud lift and shift

A quote from **ThoughtWorks Technology Radar** on [Cloud lift and shift](https://www.thoughtworks.com/radar/techniques/cloud-lift-and-shift)
*As more organizations are choosing to deploy applications in the cloud, we're regularly finding IT groups that are wastefully trying to replicate their existing data center management and security approaches in the cloud. This often comes in the form of firewalls, load balancers, network proxies, access control, security appliances and services that are extended into the cloud with minimal rethinking. We've seen organizations build their own orchestration APIs in front of the cloud providers to constrain the services that can be utilized by teams. In most cases these layers serve only to cripple the capability, taking away most of the intended benefits of moving to the cloud. In this edition of the Radar, we've chosen to rehighlight cloud lift and shift as a technique to avoid. Organizations should instead look more deeply at the intent of their existing security and operational controls, and look for alternative controls that work in the cloud without creating unnecessary constraints. Many of those controls will already exist for mature cloud providers, and teams that adopt the cloud can use native APIs for self-serve provisioning and operations.*


# Cloud native

Lifting and shifting is often an suboptimal migrations strategy simply because and architecture of the system being migrated doesn't meet certain characteristics that are required to deliver on cloud promises like costs optimization, increasede agility, resiliency etc. 
Cloud-native is an approach to building and running applications that fully exploits the advantages of the cloud computing delivery model. Very often cloud-native applications will run on a Platform-as-a-Service installation and would embrace horizontal scaling. 
Cloud-native applications are fast and stay safe and secure while being fast. They need to be able to scale easily when the success hits them which most likely means horizontal scaling. Architecting cloud-native solutions often lead to:

## microservices
Concept of microservices builds upon *Bounded Context* concept of *Domain Driven Design*. Microservices can scale horizontally idependently from other services. They support agility and enable local cost optimizations. 

## API based collaboration
**API first** approach is a follow up on microservices where the API is created first as the contract between services. Subsequently the changes inside the service are safer to the system as are not exposed beyond the api. The API could be *RESTful* but messaging is fine or even preferrable in certain scenarios.

## Self service agile infrastructure 
Docker PAAS is an example of this. Creating infrastructure on demand. This the end game of the DevOps. In this scenario the remains of central operations are just enabling all this to happen rather than provisioning boxes 

## Antifragile
Architecture that fares well under stress. Engineered for this behavior. Think Netflix "Simian Army", "Chaos Monkey" etc. 

## The Twelve-Factor Application and beyond

[The twelve-factor app](https://12factor.net/) - initially created around Heroku PaaS solution - is a methodology for building software architecture optimized for cloud adoption.
The goal was to:
- minimize time and cost for new developers joining the project.
- offer maximum portability between execution environments.
- enable continuous deployment for maximum agility.
- obviate the need for servers and systems administration. • 

Twelve-factor application can scale up without significant changes to tooling, architecture, or development practices.
The shorthand prescription to achieve that was to:
- use declarative formats for setup automation. 
- have a clean contract with the underlying operating system.
- minimize divergence between development and production

An extended version consists of twelve factors:

### Codebase
One codebase tracked in revision control, many deploys. Single code base is a unit of deployment. Once the system grows and contains multiple applications the codebase should be split to reflect that. Avoid single applications built from multiple codebases or codebases sharing code. Shared code should be explicitly built from it's own codebase and in the form of a library referenced via dependency manager. 

Deploment effectively means "running" the codebase. 

In the language of Git and Microservices one repository means one microservice.
Some would say one repository is a one process but that might be too restrictive. 


### Dependencies
Explicitly declare and isolate dependencies. Ecosystem chosen to built the system in should support package manager.
It effectively requires an ability to isolate the application from other applications, no shared dependencies (no Global Assembly Cache in MS/.NET world!).
One should be able to clone the repository, restore the dependencies and subsequently build the application and run it.

### Config
Store configuration in the environment. That means anything that varies between environments like DEV, UAT, STAGE, PROD etc.
There should be strict separateion between configuration and code. Configuration is not checked into version control system.
A litmus test to see if credentials and configuration were have been properly externalized is to imagine the
consequences of application’s source code being pushed to public GitHub.

External configuration supports ability to deploy immutable builds to multiple environments automatically via Continuous Delivery
pipelines and helps maintain development/production environment parity.
In the world of serverless environment variables might be the only thing that can be used.

As a consequence all environment-dependent entries in web.config app.configs, yamls are an anti-pattern for the cloud.

With the rapid growth of the complexity of the system certain configuraion stored in environmental variables can become hard to manage and changes can require machine reboot. 
An example of this might be addresses of backing services. Service location in a form of a tool like Consul or Zookeper can be considered in such cases.

### Backing services
Treat backing services as attached resources
In the cloud also disk and file access should be treated as a backing service.
It's best when the application can cope with attached resources evolving and fluctuating (hot swap).
Certain application architectures are more explicit in specifying boundaries and attached resources and as such are more suitable for cloud-native systems.
An example might be [Ports & Adapters](http://alistair.cockburn.us/Hexagonal+architecture) aka **Hexagonal Architecture** where adapters mark attachments of backing services. 

**Few rules for resource binding:**
- An application should declare its need for a given backing service but allow the cloud environment to perform the actual resource binding.
- The binding of an application to its backing services should be done via external configuration
- It should be possible to attach and detach backing services from an application at will, without re-deploying the application.

When managing backend services attachments useful is a concept of **circuit breakers** which among others is described in pre-cloud era book [Release It! Design and Deploy Production-Ready Software](https://pragprog.com/book/mnee/release-it). The book shows how to design and architect application for the harsh realities it will face. How to design application for maximum uptime, performance, and return on investment. [Second edition to be available early 2018](https://pragprog.com/book/mnee2/release-it-second-edition) and new coverage includes DevOps, microservices, and cloud-native architecture.

### Build, release, run
Strictly separate build and run stages. Build process simply converts an application codebase into *executable* which is not to be tinkered with afterwards - it's an immutable artifact.

### Processes
Execute the app as one or more stateless processes
Modern cloud-native applications should each consist of a single stateless processes. All long-lasting state must be external to the application, provided by backing services. So the concept isn’t that state cannot exist - it is that it cannot be maintained within your application.

Sticky sessions have no place in cloud-native application architecture. Backing store with users' sessions details is recommended.
Memory space or a file system might be used as a brief single-transaction cache. Serverless architectures won't be able to support any of the stateful designs.

### Port binding
Export services via port binding. Application should bind to a port and receive request via that port or return response through it. It doesn't have to be HTTP port, it can be some messaging middleware "port" pointing to a queue etc. Application should be self-hosted, spun up and start listening on a port.  No need to depend on special external container (don't mean Docker here - think IIS, application servers etc.). Many processes spun and listening on ports scale well and can serve large number of requests while traffic is load balanced using reverse proxy (like Nginx).

Using Docker have a separate container for each application. On container level cloud provider would assign ports and connect applications with backend services. Do not use single container for many apps. Applications where runtime port binding is possible can serve as backing services for other applications.


### Concurrency
Scale out via the process model.
It is conceivable to have multiple threads within a single process if necessary and if programming model supports that however, it's preferable to scale out using new instances of the process. Horizontal scaling whenever possible.

Different workloads can be supported by separately scaled out processes. A rule of thumb for HTTP workload might be to respond within 250ms. Anything demanding more processing time could be handled in a background process. Scaling out the top priority processes separately is possible and quite easy. 

Processes share nothing. It is good not to overuse OS-specific hosting methods. An example: Avoid making everything a **Windows Service**. .NET Core console application can be made portable and can run on Linux while TopShelf .NET service cannot.

Use backing service when process needs to cache expensive stuff. It is preferable to cache memory that has to be filled out upon restart reducing disposability. Use Redis, memcached etc.

### Disposability
Maximize robustness with fast startup and graceful shutdown.
On a cloud instance, an application’s life is as ephemeral as the infrastructure that supports it. A cloud-native application’s processes are disposable, which means they can be started or stopped rapidly. An application cannot scale, deploy, release, or recover rapidly if it cannot start rapidly and shut down gracefully. Build applications
that not only are aware of this, but also embrace it to take full advantage of the platform.
Stopping gracefuly means terminating in reasonable time frame. Decide how current work is processed and wrapped up. Finishing ongoing requests can cause problems if takes too long. Very often interruption and depending on communicating infrastructure is the best option - interrupt work and give back message to the queue so it can be processed by other process or later. Requires thinking in terms of transactions or alternatively requires an attampt at idempotency.
Elastic scaling means rapid deployment of new instances, auto registration in service discovery, having load balancer redirect traffic to new instances, shutting them down quickly, unregister etc.

### Dev/prod parity
Keep development, staging, and production as similar as possible.
There are three dimensions to this rule:

- Time - develop and deploy immediately - every commit is a candidate for deployment. Any increase in lead time increases inventory problems accross development pipeline. It's anti-agile and goes agains the cloud promise.
- People - you build it - you deploy it - you run it. Don't differentiate between people servicing different environments
- Resources - in development use the same resources, databases, services, files, the same topology, technical stack as in production to avoid surprises.

### Logs
Treat logs as event streams, that is, logs are a sequence of events emitted from an application in time-ordered sequence.
A truly cloud-native application never concerns itself with routing or storage of its output stream.
Application writes all of its log entries to *stdout* and *stderr*. This might scare a lot of people, fearing the loss of control that this
implies. Execution environment captures the stream and routes to storage. Analysis tools are used to view and analyze logs. Think stacks like ELK.

### Admin processes
Run admin/management tasks as one-off processes


Administrative tasks:
- rafactor scripts, crons and other stuff into an API (REST?) that will be secured and callable
- one time jobs can be lambas/functions not requiring provisioning of anything
 
Admin processes
- should be located along with the app - collate tools with use .NET CLI

Authentication and authorization
In an ideal world, all cloud-native applications would secure all of
their endpoints with RBAC (role-based access control).1 Every
request for an application’s resources should know who is making
the request, and the roles to which that consumer belongs

### Aftermatch
Applied to modern cloud platforms twelve-factors lead to PaaS and serverless.


# Thoughts on cloud migration
Organization willing to adopt cloud needs to make a shift towards Continuous Delivery. There is no cloud native without Continuous Delivery.
Cloud migration enables also organizational migration from centralized governance towards decentralized autonomy (somewhat tricky for us since Cloud Working Group is a form of "centralized governence")

Enterprises normally adopt centralized governance structures around application architecture and data management, with committees responsible for maintaining guidelines and standards, as well as approving individual designs and changes. Adoption of cloud-native application architectures is almost always coupled with a move to decentralized governance. The teams building cloud-native applications ("Business Capability Teams") own all facets of the capability they’re charged with delivering. They own and govern the data, the technology stack, the application architecture, the design of individual components, and the API contract delivered to the remainder of the organization. If a decision needs to be made, it’s made and executed upon autonomously by the team. 

**Inverse Conway Maneuver** - rather than build an architecture that matches company's organizational chart, the company can determine the architecture they want, and restructure their organization to match that architecture. Once done, according to Conway, the architecture desired will eventually emerge.
What remains then is to determine what teams to create. If we follow the Inverse Conway Maneuver, we’ll start with the domain model for the organization, and seek to identify business capabilities that can be encapsulated within bounded contexts. Once we identify these capabilities, we create business capability teams to own them throughout their useful lifecycle. Business capability teams own the entire development-to-operations lifecycle for their applications.

### Decomposing monoliths
Traditional n-tier, monolithic enterprise applications rarely operate well when deployed to cloud infrastructure, as they often make unsupportable assumptions about their deployment environment that cloud infrastructures simply cannot provide. A few examples include: Access to mounted, shared filesystems Peer-to-peer application server clustering,shared libraries Configuration files sitting in well-known locations. Monoliths couple change cycles together such that independent business capabilities cannot be deployed as required, preventing speed of innovation. Services embedded in monoliths cannot be scaled independently of other services, so load is far more difficult to account for efficiently. 
Developers new to the organization must acclimate to a new team, often learn a new business domain, and become familiar with an extremely large codebase all at once. This only adds to the typical 3– 6 month ramp up time before achieving real productivity. 
Attempting to scale the development organization by adding more people further crowds the sandbox, adding expensive coordination and communication overhead. 
Technical stacks are committed to for the long term. Introducing new technology is considered too risky, as it can adversely affect the entire monolith.
It’s not enough to decompose monolithic applications into microservices. Data models must also be decoupled. If business capability teams are supposedly autonomous but are forced to collaborate via a single data store, the monolithic barrier to innovation is simply relocated.
We couple bounded contexts with the database per service pattern, where each microservice encapsulates, governs, and protects its own domain model and persistent store.

 
# Useful links

- [Cloud Native Computing Foundation](https://www.cncf.io/)


# Be aware of political turmoil around the cloud
- "Anyone but AWS" club
- AWS threatened by Kubernetes
- Open Container Initiative (against Docker monopoly)