GCP Associate Engineer Certification
====================================

- [Best Practices](#best-practices)
- [Logging](#logging)
- [Roles And Security](#roles-and-security)
- [Billing and Pricing](#billing-and-pricing)
- [Cloud Storage and Databases](#cloud-storage-and-databases)
- [Compute Engine](#compute-engine)
- [Kubernetes Engine](#kubernetes-engine)
- [Machine Types](#machine-types)
- [Misc Service Related](#misc-service-related)
- [Command Line](#command-line)

## BEST PRACTICES

You are onboarding a new employee. What is a Google-recommended practice providing them necessary GCP access?
* Add them to a Google Group for their job function.
  * Access to GCE instances should not be managed directly between instances and employees; a Google Group should be used to manage this.

You need to make sure a GCE instance can access other services in GCP.  What is a Google-recommended practice for enabling this?
* Grant a service account access to the required resources.
* Associate the service account to the instance.
  * When you create an instance, you can specify which service account the instance will use.

You are responsible for securely managing employee access to Google Cloud.  What are two Google-recommended practices for this?
* Use Cloud Identity or G Suite to manage Google accounts for employees.
* Enforce MFA on employee accounts.

## LOGGING

You are designing the logging structure for a **containerized** Java application that will run on GKE. What is the recommended practice that will use the least number of steps to enable your developers to later access and search logs?
* Have the devs write log lines to stdout and stderr
  * “Stackdriver Logging is enabled by default when you create a new cluster using the gcloud command-line tool or Google Cloud Platform Console.” Logging to stdout and stderr on GKE *is* the recommended way to log: “Containers offer an easy and standardized way to handle logs because you can write them to stdout and stderr. Docker captures these log lines and allows you to access them by using the docker logs command. As an application developer, you don't need to implement advanced logging mechanisms. Use the native logging mechanisms instead.”

You are designing the logging structure for a **NON-containerized* Java application that will run on GAE. What is the recommended practice that will use the least number of steps to enable your developers to later access and search logs?
* Have the devs write logs using the App Engine Java SDK
  * In App Engine Standard, you should log using the App Engine SDK and the connection to Stackdriver.

You are designing the logging structure for a **NON-containerized** Java application that will run on GCE. What is the recommended practice that will use the least number of steps to enable your developers to later access and search logs?
* Have the devs write log lines to a file named application.log and install the Stackdriver agent on the VMs, configure the Stackdriver agent to monitor and push application.log.
  * Configuring a custom log file location is the recommended way to get logs to Stackdriver for [non-containerized] apps on GCE.

You are designing the logging structure for a **containerized** Java application that will run on GAE Flex. What is the recommended practice that will use the least number of steps to enable your developers to later access and search logs?
* Have the devs write log lines to stdout and stderr
  * In App Engine Flex the connection to Stackdriver is handled automatically for you. In GAE Flex, you *could* write logs using the App Engine SDK--and that would work--but it’s best practice for containers to log to stdout and stderr, instead.

You are planning a log analysis system to be deployed on GCP. What would be the best way to store the logs, long-term?
* Cloud Storage
  * Stackdriver Logging only retains logs for 30 days, but it can then send the logs on to GCS for long-term storage (in whichever storage class is desired).

You need to view both request and application logs for your Python-based App Engine app. What is the best way to do this?
* Use built-in support to get both request and app logs to Stackdriver.
  * GAE natively connects to Stackdriver and sends any request & app logs you give it, via the GAE SDK.

You are planning a log analysis system to be deployed on GCP. What is the best way to ingest the logs?
* Stackdriver Logging
  * Stackdriver is perfect as the *first* ingestion point for accepting many logs. It can send logs to Cloud Storage for archiving, or to Pub/Sub for streaming to something else like Dataflow for processing.

You are planning a log analysis system to be deployed on GCP. What is the best service for processing streamed logs?
* Dataflow

You are planning out your usage of GCP. What is something you need to consider about default Service Accounts?
* Default service accounts are restricted in what they can do by the default access scopes.

You need to determine who just started a particular GCE instance that does not meet your organization’s resource labelling policies. How can you determine who to follow up with, in the least number of steps?
* From the notifications menu, navigate to the Activity Log. Look for a log line like “USER_EMAIL created INSTANCE_NAME”.
  * The Compute Engine section of the console does not identify any “Owner”. Information about “Who did What, and When?” should be identified from the Activity Log or fuller Audit Log. It takes fewer steps to get to the Activity Log by dropping the notifications menu (though navigating from the project dashboard is also a valid way to get there). Because the instance was just started, it should show up on the first page without needing to change the displayed date range.

You navigate to the Activity Log for a project containing a GKE cluster you created.  If you filter the Resource Type to “GCE VM Instance”, what log lines will you see?
* Log lines for GKE node creation will show up in the activity log.
  * But the creation is not attached to your account--you only created the GKE cluster. Neither is it the GCE default service account that creates such instances--that account is meant to be used by applications running *on* GCE instances, not GKE management like this. Instead, log lines will use the passive voice “GKE_NODE_INSTANCE_NAME was created” to indicate that this was an automatic action taken by GCP because you had previously configured/requested it do that.

You go to the Activity Log to look at the “Create VM” event for a GCE instance you just created.  You set the Resource Type to “GCE VM Instance”.  Which activity type filter will display the “Create VM” event you wish to see?
* Setting the Activity Types dropdown to “Configuration”
  * “Create VM” is considered to be a “Configuration” activity

Your security team wants to be able to audit network traffic inside of your network. What's the best way to ensure they have access to the data they need?
* Enable flow logs

## ROLES AND SECURITY

Which role has the highest level of access?
* Project Owner
  * (There is also a “Super Admin” which has equal access to everything if granted)

You are designing the object security structure for sensitive customer information. What should you be sure to include in your planning?
* Assign limited access, to achieve least privilege.
* OR, if you do not grant any bucket-level permissions, new objects would be secure by default.
  * Least privilege is a paramount concern for data security, and you definitely do want to restrict access as much as possible to support this.

A co-worker tried to access the `myfile` file that you have stored in the `mybucket` GCS bucket, but they were denied access. What is the best/quickest way to allow them to view it?
* In Cloud Shell => `gsutil acl ch -u coworker@email.domain:r gs://mybucket/myfile`

You need to list objects in a newly-created GCS bucket. What is the least permissive role that would allow you to do this?
* roles/storage.legacyBucketReader

Can you generate access keys for service accounts? 
* Yes. You may generate a small number of keys per service account to facilitate key rotation.
  * It is best to let Google manage all service account keys, but it is possible to generate some so you can use them outside of the situations that GCP handles--such as from AWS or your local machine. You don’t need to remember how many keys you can generate (10), only that the reason you can create more than one is so that you can put a new one in place before disabling an old one (i.e. key rotation).

Concerning GCE service accounts:
  * If you don't do (or specify) anything, the default service account will be attached by default to each new GCE instance.
  * However, you can stop that from happening by either deleting the default service account or opting out of attaching it when you are creating a new GCE instance.
  * Service accounts are meant to be used by programs and they are one--but not the only--way to manage access to resources.

How should you enable a GCE instance to read files from a bucket in the same project?
* When launching the instance, remove the default service account to it falls back to project-level access.
* By default, both the default service account and the default scopes can read from GCS buckets in the same project, so you should just leave those alone and it will work.

You have a GCE instance using the default service account and access scopes allowing full access to storage, compute, and billing.  What will happen if an attacker compromises this instance and runs their own program on it?
* The token will only allow the attacker (as any user) to perform whatever is allowed by *both* the service account *and* the access scopes. Since both the service account and the access scopes are missing some capabilities from the other, the actual access possible by using the token will be less than either of them, independently.

You are working together with a contractor from the Acme company and you need to allow **App Engine** running in one of Acme’s GCP projects to write to a Cloud Pub/Sub topic you own. What pieces of information are enough to let you enable that access?
* The email address of the Acme service account
* The Acme GCP project’s project ID
  * You need to grant access to the service account being used by Acme’s App Engine app (so you need the email address of the project’s service account, which would take the form: PROJECT_ID@appspot.gserviceaccount.com)

You are working together with a contractor from the Acme company and you need to allow **GCE instances** running in one of Acme’s GCP projects to write to a Cloud Pub/Sub topic you own. What pieces of information are enough to let you enable that access?
* The email address of the Acme service account
* The Acme GCP project’s project number

Enable a GCE instance in Project A to read files from a bucket in Project B:
* Since the default scopes allow reading from GCS, all that remains to get this situation working is for Project B (which owns and controls access to the bucket) to grant access to the service account being used. APIs are enabled for a project and do not differ between internal and external access.
* Do not change the default service account setup and attachment.
* In Project B, grant bucket access to Project A’s default compute service account.

You're migrating an application to GCP. The security team wants detailed *visibility* of all projects in the ORG. You provision the Google Cloud Resource Manager and set up yourself as the org admin. What Google Cloud Identity and Access Management (Cloud IAM) roles should you give to the security team?
* Org Viewer and Project Viewer
  * Follow the Principle of Least Privilege

You've setup and tested several custom roles in your development project. What is the fastest way to create the same roles for your new production project
*	Use the `gcloud iam roles copy` command and set the destination project
	* (Can be used to copy roles to different organizations or projects)

## BILLING AND PRICING

How can you link a new project with your billing account?
* If you created the project in the console, do nothing.
* If you created the project via gcloud, link it with a command under gcloud beta billing.

BigTable is priced by provisioned nodes. BigTable does not autoscale. To use the pricing calculator for BigTable, you need to enter _____
* The number of BigTable nodes you’ll provision

Who can change the billing account linked to a project?
* Project owner
* Billing administrator

You are planning to use GPUs for your system on GCP. In what tabs of the pricing calculator are GPUs relevant?
* GPUs can be entered on both the GCE and GKE tabs.
  * GPUs are found here because they can be added to VM instances to accelerate graphics-intensive workloads, and since GKE uses GCE under the hood, GPUs are available under both tabs in the pricing calculator.

(In the pricing calculator, prices correspond to whatever region you select for each resource. Custom Machine Types are fully supported. You can choose whichever currency you’d like for the the estimate. Sustained Use Discounts are fully supported.)

You need to estimate the pricing of a **new** system you are building. You should best use the:
* Pricing Calculator

You need to visualize your spend over time for a **currently running system** on GCP. You should use:
* Data Studio

You are planning to use BigQuery for a system you will manage. How you will use the pricing calculator?
* You have to separately estimate the data to be stored, streamed, and queried, by the system and enter your estimated amounts into the GCP pricing calculator.

You are planning to host your system in Google App Engine. What is something you App Engine **cannot** do, and so isn't something you'd see while using the pricing calculator?
* The OS on App Engine isn't something you can manage, so that would not appear in the pricing calculator.

How should you estimate the annual cost of running a BigQuery query that is scheduled to run NIGHTLY:
* Before running queries, preview them to estimate costs with:
	* `bq query --dry_run`
	* The query returns the bytes read (time isn't important) which is used with the Pricing Calculator to estimate cost.

You're using a self-serve Billing Account to pay for 2 projects. Your billing threshold is set to $1000 and between the 2 projects, you're spending roughly $50/day. It has been 18 days since you were last charged. When will you be charged next?
* In 2 days
  * Billing is either monthly, or the at the threshold, whichever comes first, and in this case the threshold is hit before the end-of-month.


## CLOUD STORAGE AND DATABASES

You want to create a new GCS bucket in Iowa. How could you go about doing this?
* GCP does not have top-level location selectors like AWS does. GCP is global by default and it’s only individual resources that live in certain locations. When you’re creating a bucket in the console, the wizard asks you where you want to put it. Also, you cannot move a bucket after its region has been chosen during creation.
  * So, you would simply begin creating the bucket and set the location to Iowa when prompted.

You have a volume of data that is accessed very rarely (on average once every 3-4 years) but should be retrieved very quickly (less than one second) when it is. What do you need to consider when deciding how to store this data?
* ALL storage classes offer low latency (time to first byte typically tens of milliseconds) and high durability.
  * So, for this case, Coldline would be the best option for the price, and will still provide low latency during data retrieval.

You need to store thousands of 2TB objects for one month and it is very unlikely that you will need to retrieve any of them. What option would be the most cost-effective?
* Nearline
  * Because, Coldline’s minimum storage duration of 90 days makes it more expensive than Nearline. Multi-Regional and Regional are both more expensive than Nearline as well.

You need to store **trillions** of 2KB objects for one month and it you will need to run *analytical processing* against all of them from hundreds of nodes. What database option would be the most cost-effective?
* BigTable
  * This is made for large analytical workloads. Cloud Storage makes you pay for read operations, so that can get expensive if it’s not the right fit.

You need to store thousands of 2TB objects for **one week** and it is very unlikely that you will need to retrieve any of them. What option would be the most cost-effective?
* Regional Cloud Storage
  * As a comparison, Bigtable is not for large objects; Nearline/Coldine have minimum storage durations and are more expensive; Multi regional is more expensive.

You need to store a large amount of unstructured data, including video, audio, image, and text files, with little management.
* Cloud Storage
  * Without knowing retrieval requirements or storage duration to determine the best storage class, GCS in the general sense is the best answer.

You currently have 850TB of Closed-Circuit Television (CCTV) capture data and are adding new data at a rate of 80TB/month.  The rate that data is captured and stored is expected to grow to 200TB/month within one year because new locations are being added, each with 4-10 cameras. What GCS bucket storage strategy best suits this purpose without encountering storage or throughput limits?
* One Cloud Storage bucket for all objects
  * You don’t need to split your data up to avoid bucket-level limits. It is generally easiest (and best) to manage all your data in a single bucket and using things like folders for organizing them. In fact, if you separate data into many buckets, you are more likely to encounter limits around bucket creation and deletion. 

You currently have 850TB of Closed-Circuit Television (CCTV) capture data and are adding new data at a rate of 80TB/month. The rate that data is captured and stored is expected to grow to 200TB/month within one year because new locations are being added, each with 4-10 cameras.  **Archival data must be stored indefinitely, and as inexpensively as possible.**  The users of your system currently need to access 60GB of *current-month* footage and 50GB of archival footage, and access rates are expected to grow linearly with data volume. What storage option best suits this purpose?
  * Just use Coldline for everything.
    * Data cannot be transitioned from Multi-Regional to Regional through Lifecycle Management; that would change the location. The access rate for new data is 60/80000--so very low--and archival data access is even lower (50/850000). Because of this, the most cost-effective option is also the simplest one: Just use Coldline for everything.

You need to store some recently recorded customer focus sessions into a new GCP project. How can you enable the GCS API in the fewest number of steps?
* Do nothing. It’s enabled by default. Ha.
  * But this isn't true for all APIs, so watch out.

You need to store some *structured data* and query and continually update it with SQL from your web app backend.  The data volume and query load are reasonably consistent and you would like to **reduce ongoing maintenance and management costs.**  Which option would best serve these requirements?
* Cloud SQL - a managed SQL service

You need to set up Lifecycle Management on a new Cloud Storage bucket. If you have existing buckets, What should you consider before creating the new storage bucket?
* Identify an existing bucket with the configuration you want, download that bucket’s configuration JSON using the JSON api, and apply the configuration file for the new bucket via gsutil.

For websites deployed to Cloud Storage that are going through frequent critical changes, implementing a back-up/roll-back plan is simple by:
*	Enabling object versioning on the website’s static data files stored in Google Cloud Storage

## COMPUTE ENGINE

You are thinking through all the things that happen when a Compute Engine instance starts up with a startup script that installs the Stackdriver agent and runs gsutil to retrieve a large amount of data from Cloud Storage. What are the steps that occur throughout this process?
* Whether or not it is the default service account, (1) the service account must exist before it can be attached to the instance. After a request to create a new instance has been accepted and while space is being found on some host machine, (2) that instance starts in the Provisioning state. After (3) space has been found and reserved on a host machine, (4) the instance state goes to Staging while the host prepares to run it and sorts out things like the network adapter that will be used. (5) Immediately when the VM is powered on and the OS starts booting up, the instance is considered to be Running. That's when (6) gcloud completes, if it was run without `--async`.

You need to start a set of virtual machines to run year-end processing in a new GCP project.  How can you enable the Compute API in the fewest number of steps?
* Each API must be enabled before it can be used. Some APIs are enabled by default, but GCE is not.
* Navigating to the Compute Engine of the console automatically enables the GCE API. You do not have to configure authentication to be able to use Cloud Shell, but regardless, using Cloud Shell would take more steps than simply navigating to the GCE console.

In Cloud Shell, you run the command `gcloud compute instances list`, and the response that you see is `HTTPError 403: Access Not Configured.`. What is a likely explanation for this error message?
* The GCE API has not yet been enabled for this project.

You need to host a legacy accounting process on **SUSE Linux Enterprise Server 12 SP3.**  Which of the following is the best option for this?
* GCE
  * (You cannot choose the OS through other services like App Engine or Kubernetes Engine)

## KUBERNETES ENGINE

You are planning to use Persistent Disks in your system.  In the context of what other GCP service(s) will you be using these Persistent Disks?
* GCE and GKE
  * Persistent disks attach to compute engine instances, used by GKE.

You have two web applications that you want to deploy in GCP--one written in Ruby and the other written in Rust. What two GCP services would be capable of handling these apps?
* Compute Engine
* Kubernetes Engine
  * (Ruby and Rust applications could both be run in containers or directly on VMs.)

You run the command `kubectl describe pod mypodname` in Cloud Shell. What should you expect to see?
* Information about the named pod

What will happen if a running GKE **pod** encounters a fatal error?
* GKE tries to ensure that the number of pods you’ve specified in your deployment are always running, so it will restart one on an available node if it fails.
  * Pods do not 'heal' or repair themselves. For example, if a Pod is scheduled on a node which later fails, the Pod is deleted. Similarly, if a Pod is evicted from a node for any reason, the Pod does not replace itself.

You have a GKE cluster that currently has six nodes but will soon run out of capacity. What should you do?
* In the GKE console, edit the cluster and specify the new desired size.
  * (GKE Clusters are editable, not immutable, and should not be recreated because of changes in demand. Cluster autoscaling is an optional setting. You do not manage nodes via GCE, directly--you always manage them through GKE, even though you can see them via GCE.)

You are planning to run a multi-node database on GKE. What do you need to consider?
* Kubernetes StatefulSet objects exist to manage applications that **do** want to preserve state--unlike the normal applications that should be stateless.

What will happen if a running GKE **Deployment** encounters a fatal error?
* GKE Deployments are a declaration of what you want. GKE Deployments are configuration information and **do not directly encounter fatal errors.**

What will happen if a running GKE **node** encounters a fatal error?
* GKE will automatically start that node on an available GCE host.
* Nodes are GCE instances managed by the GKE system. If one of the nodes dies, GKE will bring another node up to replace it and will ensure that any affected pods are restarted.

A pod is being terminiated in a GKE cluster. What happens?
* The memory used by the containers will be freed.

You have a GKE cluster that currently has six nodes but has lots of idle capacity.  What should you do?
* `gcloud container clusters resize mycluster --size=5`
  * Clusters are editable and should be recreated due to changes in demand. Autoscaling is optional.
  * You never manage nodes directly in GCE — always through GKE (but you can SEE them in GCE).

You have a GKE cluster that has fluctuating load over the course of each day and you would like to reduce costs.  What should you do?
* Edit the cluster to enable autoscaling.

Your company needs to create a new Kubernetes Cluster on Google Cloud Platform. As a security requirement, they want to upgrade the nodes to the latest stable version of Kubernetes with no manual intervention. How should the Kubernetes cluster be configured?
*	Enable node auto-upgrades

Your company needs to create a new Kubernetes Cluster on Google Cloud Platform. They want the nodes to be configured for resiliency and high availability with no manual intervention. How should the Kubernetes cluster be configured?
*	Enable auto-repairing for the nodes
	* (Auto-upgrades are to upgrade the node version to the latest stable Kubernetes version)

Your team uses a third-party monitoring solution. They've asked you to deploy it to all nodes in your Kubernetes Engine Cluster. What's the best way to do that?
* Deploy the monitoring tool as a DaemonSet.
	* “Daemon set” helps deploy applications or tools that you need to run on all nodes.
	* StatefulSet is useful for maintaining state — a set of pods with unique persistent identities and stable hostnames.


## MACHINE TYPES

The number at the end of the machine type indicates how many CPUs it has, and the type tells you where in the range of allowable RAM that machine falls--from minimum (highcpu) to balanced (standard) to maximum (highmem). The cost of each machine type is determined by how much CPU and RAM it uses. Some examples:
* n1-standard-8
* n1-highcpu-8
* n1-highmem-16
  * The n1-highmem-16 has twice as many CPUs as the n1-highcpu-8
  * The n1-highcpu-8 would be least expensive
  * You could image that “highcpu” also means “lowmemory” comparatively speaking
  * (Check docs for machine type specifications)

You are currently using an `n1-highcpu-8` machine type and it is good but you would like **just a bit** more RAM.  Which of the following is the most cost effective option to achieve this?
* The custom machine type with 8 CPUs is by far the best choice, here. You don’t want (or need) to choose a machine type with a different number of CPUs, so the only other option to consider would be `n1-highmem-8`--but that has the *maximum* amount of RAM for 8 CPUs and you only want (and only want to pay for) “just a bit more RAM”.

Instance templates are immutable, you never edit them.

## MISC SERVICE RELATED

Google has just released a new "XYZ" service and you would like to try it out in your pre-existing project. How can you enable the XYZ API in the fewest number of steps?
* Open cloud shell, run `gcloud services enable xyz.googleapis.com`

You need to process batch data in GCP and reuse your existing Hadoop-based processing code. What is a managed service that would best handle this situation?
* Cloud Dataproc
  * Made for running Hadoop/Spark work.

App Engine Standard supports apps written in Java, Python, and Go. Cloud Dataflow and Cloud Dataproc are services for processing large volumes of data, not for hosting web apps. Ruby and Rust applications could both be run in containers on App Engine Flexible.

You need to process data streamed from Cloud Pub/Sub. What is the best managed service that would handle this situation?
* Cloud Dataflow

A company wants to store images in a Cloud Storage bucket and generate thumbnails as well resize the images. They want a managed service which will help them scale automatically from zero to scale and back to zero. What should you use?
*	Cloud Functions
	* Automatically scales as per the demand, with no invocations if there is no demand.
	* Serverless
	* (Compute Engine, Kubernetes Engine, and App Engine need configuration to scale down, and warm up time to scale up)

A company is planning to migrate a web application to Google App Engine. However, they’ll still continue to use their on-premises database. How can they setup the application?
* App Engine Flexible Environment, using Cloud VPN to connect to the DB
	* App Engine provides on-prem connectivity
	* App Engine *Standard* CANNOT use Cloud VPN

**When to choose the standard environment:**
*Application instances run in a sandbox, using the runtime environment of a supported language.*
*Applications that need to deal with rapid scaling.*
*Experiences sudden and extreme spikes of traffic which require immediate scaling.*
*Cannot use Cloud VPN*

**When to choose the flexible environment:**
*Application instances run within Docker containers on Compute Engine virtual machines (VM).*
*Applications that receive consistent traffic, experience regular traffic fluctuations, or meet the parameters for scaling up and down gradually.*
* Can use Cloud VPN*

You're using Cloud SQL to host critical data. To enable high availability in case a complete zone goes down, how should you configure Cloud SQL?
*	Create a Failover replica in the same region, but a different zone
	* This rovides high availability
	* The Failover MUST be in the same region as the primary
	* (A “Read Replica” is wrong as it doesn’t provide failover, only additional read capacity)

A marketing application has been running on App Engine for a few weeks with Autoscaling, and it's been performing well. But, the team is planning on a massive campaign, and they expect a lot of burst traffic. How can they ensure there are always 3 idle instances ready?
*	Set the `min_idle_instances` property in the app.yaml to 3

You're slowly rolling out new functionality to monitor for errors. They contain some big changes to the UI. You've chosen to use traffic splitting to perform a canary deployment. You're going to start by rolling out the code to just 15% of your users. How should you go about setting up traffic splitting with the user getting a consistent experience?
*	Deploy the new version using the `no-promote` flag. Split the traffic using Cookie.
	* `--no-promote` parameter avoids the new version getting 100% of the traffic. Once it’s deployed and tested, Cookie can split and keep traffic UX consistent.

You've created a new firewall rule to allow incoming traffic on port 22, using a target tag of "dev-ssh". You tried to connect to one of your instances, and you're still unable to connect. What steps do you need to take to resolve the problem?
*	First, the firewall needs to be associated with the instance for the instance to follow the firewall rules, so:
	* Apply a network tag of “dev-ssh” to the instance you’re trying to connect into and try again.

Your company wants to setup Production and Test environments. They want to use different subnets and the key requirement is that the VMs must be able to communicate with each other using internal IPs no additional routes configured. How can the solution be designed?
*	Configure a single VPC with 2 subnets having the different	CIDR range hosted in the different region.
  * VMs need to be able to communicate using private IPs that should be hosted in the same VPC. The Subnets can be in any region, however they should have non-overlapping CIDR range.

The Cloud Launcher was renamed to be the GCP Marketplace--so these refer to the same thing--and this is a quick way to deploy all sorts of different systems, including Nginx. So if you're asked, "What is the fastest way possible..." or "the easiest way to quickly setup..." some service, the answer is likely to be, or involve, Cloud Launcher or GCP Marketplace.

## COMMAND LINE

You have just installed the Google Cloud SDK. How do you initialize the command line tools?
* `gcloud init`

You are currently creating instances with `gcloud compute instances create myvm --machine-type=n1-highmem-8`.  This is good but you would like **just a bit more** RAM. What command to create the new instance would be the most cost effective?
* `gcloud compute instances create myvm --custom-cpu=10 --custom-memory=60`
  * Not using a custom machine type, and bumping up to the next level CPU count, or memory capacity, would result in overshooting your requirements and cost more than necessary. A custom machine is best to keep costs down.

whats the easiest way to delete a project?
* `gcloud projects delete oldprojectid`

You already installed and configured `gcloud` for use on your work computer (not Cloud Shell). What do you need to so you can also use `gsutil` and `bq`?
* Nothing! Kind of a trick question. These tools all share their configuration, managed by gcloud.

Projects are global and and are not “located” in any region or zone. The gcloud family of tools save their default zone information locally where they’re installed, and these are separate from console settings. So, if you run the command `gcloud compute instances create newvm` but it does not prompt you to specify a zone, why might that be?
* It’s because your gcloud configuration probably includes a value for a compute/zone already.

Setting roles via command line:
* gcloud [GROUP] add-iam-policy-binding [RESOURCE-NAME] —role [ROLE-ID-TO-GRANT] —member user: [USER-EMAIL]
* In GCE context:
  * gcloud beta compute instances add-iam-policy-binding myhappyvm --role roles/compute.instanceAdmin --member user:me@example.com
