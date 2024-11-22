to track logs 
oc edit authentication.operator.openshift.io
spec:
loglevel=debug from normal
managementstate=managed


openshift upgrade
==================
3 way we can upgrade
cli
webconsole-->Administration-cluster setting--select version
console.redhat.com/openshift-->cluster list,(need to register for support require from redhat)--if register then we can upgrade and its have connection with internet
4 types of channel
stable will use 99% in every company
1. candidate-->used for testing latest cluster ,if any issue came redhat wont support (once same version available in stable or fast then they will support)
2.fast-->if redhat announce today, we will get in fast, if any issue came redhat will support 
3.stable-->deploy and update cluster and get support from redhat and work as real product
4.eus-->extended update support,if you have critical cluster,if you dont upgrade latest,so we have to extend support from redhat (for any reason like to migrate to kubernate and they plan to deco)

in openshift upgrade graph we have to check we can upgrade from 4.13.2 to 4.13.52
oc get co,oc get no,oc get mcp(machine config pool)
check cluster operater healty or not, etcd,machine-config if have issue dont plan for upgrade first need to fix that issue

before upgade need to take backup of etcd(/usr/loca/bin/cluster-backup.sh,cluster-resore.sh)
update options
1.full cluster update--all nodes will update
2.partial cluster update--pause worked node
sequence
===
first cluster operator next master node next worker node will update
in cluster operator first etcd,kube-apiserver lastone machine-config

issues:
====
we didn't get any issue on cluster operater but we will get issue with worker node
1.if worker node not change status  as expect within 30 min then we will drain worker node(pod distribution budget)

using ansible automation script we have to do rhel and windows


cronjob==========
by default its store 3 pods  history
parallelism means we have specify how many job  execute at given time
if user ask need 10 pods history -->need to add successfulyjobhistorylimit and failed jobhistorylimit

brew install watch

first create cluster role next need to bind with help of clusterrolebind  to sa

scc--security context contraints
==============================

role, role binding
cluster role, cluster role binding--
scc-->service account-->pod
sa--this type of user, if application require additional permission(scc cant bind directly to application) then scc  bind to sa and sa will bind to application then application will get permission


3 types of sa bydefault 1.builder,2.default,3.deployer

is there any option to findout what are roles assigned to service account in openshift(need to check)
oc describe serviceaccount <service_account_name>,oc get rolebinding -n <project_name> | grep <service_account_name>

oc adm policy add-scc-to-user anyuid -z Sa
for newjoiner will give some access to monitor cluster
========================
oc adm policy add-cluster-role-to-user cluster-admin -z newjoiner(oc describe sa newjoiner-->will see token id,oc get secret,oc describe secret secretname,oc login --token=code;oc whoami

1. Builder Service Account
Purpose: Used to build images from source code.

Roles: Typically has the necessary permissions to read source code repositories, pull base images, and push built images to an image registry.

Usage: Automatically used by OpenShift build configurations.

2. Default Service Account
Purpose: Acts as the default service account for pods that do not specify a specific service account.

Roles: Has basic permissions to interact with the API server, access secrets, and communicate within the cluster.
Usage: Used by default when no other service account is specified in a pod's configuration.

3. Deployer Service Account
Purpose: Manages the deployment process of applications.

Roles: Typically has permissions to create, update, and delete resources required for deployment, such as pods, services, and replication controllers.

Usage: Automatically used by OpenShift deployment configurations.
oc set sa deploy/httpd demo


chapter1 application deply
===========
oc get no or kc get no

declarative vs imperative
yaml file      -command
oc new-app httpd
oc new-app https//githuburl --name http
oc expose service httpd
curl routeurl
if future need to migrate from openshift to kubernate or kubernate to openshift we have to create application like below 
oc or kubectl create deployment name  --image=nginx(--image docker.io/path) --port=8080 --dry-run=client -oyaml

inside directory and subdirectory have some yaml,if need to create application for all directory then go oc create  -R -f demo
if code available in internate(oc apply -f http://10.9/)
if we delete name space like openshift-dns it will recreate (self healing)
(oadp,volero for backup in openshift)

if we want create same app as another namespace then get code like oc get deploy/httpd -oyaml >demo.yml

if you want modify live app dont go direct oc edit deploy,first generate yml file from deploy and modify as requirement then verify what are will reflect oc diff -f demo.yml(- means will remove,+ means will add

overlay
===============

This setup allows you to manage different environments effectively, ensuring that each environment only applies the necessary changes while keeping the core configuration consistent.

Using overlays allows you to apply specific customizations to the base configuration for each environment such as resource limits, environment variables, and replicas, without duplicating the entire configuration file. It streamlines the process of deploying to multiple environments with different requirements.

Base Configuration: Define common resources such as Deployments, Services, and ConfigMaps in a base directory.

Overlay for Development: Create an overlay directory for development with a kustomization.yaml file that includes patches specific to the development environment.

Overlay for Production: Similarly, create an overlay for production with its own kustomization.yaml file and patches.


Base
Definition: The base is a set of common, reusable configurations that serve as the foundation for different environments.

Purpose: Ensures consistency across environments by defining standard settings and resources.

Example: A base configuration might include Deployment and Service manifests that apply to all environments.

Overlay
Definition: An overlay is an environment-specific customization that modifies the base configuration to suit different needs (e.g., development, staging, production).

Purpose: Allows you to adapt the base configuration without duplicating files, making it easier to manage differences between environments.
Example: An overlay for a development environment might change resource limits, replicas, or environment variables.

Kustomization
Definition: Kustomization refers to the process of using a kustomization.yaml file to manage and apply overlays to the base configuration.

Purpose: Provides a declarative way to combine and customize configurations, supporting GitOps workflows and enhancing automation.

Example: The kustomization.yaml file specifies resources, patches, and overlays to be applied.


Base Configuration (base/kustomization.yaml)
yaml
resources:
  - deployment.yaml
  - service.yaml
Development Overlay (overlays/dev/kustomization.yaml)
yaml
bases:
  - ../../base
resources:
  - patch.yaml

Production Overlay (overlays/prod/kustomization.yaml)
yaml
bases:
  - ../../base
patches:
  - path: patch.yaml

oc kustomize base/overlay/dev -->to check syntax
oc apply -k overlay/product/


helm
============
its package management and security wise its good 

Why Use Helm in OpenShift?
Simplified Deployment: Helm automates the deployment process, making it easier to install, upgrade, and manage applications.

Reusable Packages: Helm charts can be shared and reused across different environments, promoting consistency and reducing duplication.

Version Control: Helm allows you to manage different versions of your applications, making it easier to roll back to previous versions if needed.

Community Support: There is a large repository of Helm charts available, contributed by the community, which you can use and customize.

Integration with CI/CD: Helm integrates well with CI/CD pipelines, enabling automated deployments and consistent configuration management.

Example Use Case
Imagine you have an application that needs to be deployed in both development and production environments. With Helm, you can create a Helm chart for your application, define the necessary Kubernetes resources, and package it into a chart. You can then deploy this chart to both environments using Helm commands, ensuring that the configuration remains consistent and manageable.


steps
======helm image registry hub  bitnami and artifacthub
1.install helm chart
2.add repo (helm repo add bitnami https://
3.helm install mygnix bitnam/ngix --version=18.2.0(this one latest chart) -n demo
if require take code and install in another env
help show value repo/ngix >demo and modify as your wish
helm install mygnix bitnam/ngix -f demo.yml


image stream
===========
after app created image will push to local repository why bcz after pod restarted thats image will take from local repository(it wont go to GitLab to get frequently for image stream it )


template chapter3
==================
its reusable ,we have code already, just we have to pass variable parameter values and use for deploy application its have in openshift name space
oc describe template.

Predefined Configurations: Templates allow you to define a complete set of resources (pods, services, routes, etc.) in a single file, simplifying the deployment process.

Reusable: Once created, templates can be reused across different projects and environments, ensuring consistency and saving time.

Parameterization:

Customizable: Templates support parameters, allowing you to customize aspects of your deployment without altering the underlying template. You can define variables for things like image tags, resource limits, and environment-specific settings.

oc process --parameters MySQL -n openshift
oc get template templatename -oyml -n openshift
oc new-app MySQL -p 
process is used to get parameters or create application

user authentication
=================
default user after installe cluster
1.kubeadmin-->once system is good we can login with--kubeadmin-password
2.system:admin --->first priority--system is not good we can login and fix issue-->kubeconfig(its have server detailes and cert,if not login with admin then we have to destroy there is no option to fix it)

primary resources to authentication
user --user1 user2
identity --any data of authentication are stored on identity--htpasswd,ldap
service account
group
role
openshift have two methods to authentication request
1.oauth access token
2,client cert
authentication operator-->ldap,htpasswd


namespace->openshift-authentication,co->authentication

steps===ldap,ipa,ad
first take maintenance window it will impact if we add 
1.first install htpassword if not available(yum install -y httpd-tools)
2.create user and password with help of htpasswd
3.create secret under openshift-config(critical data will store here)
4.create oauth file and add secrete name and identinty provider name (any name like swami) in that yml and run oc apply -f oath.yml then pods recreate on openshift-authentication at same time authentication cluster operater  will interrupt so we cant login at that time

open webconsole -->it will show swami name for login(swami is identity here)

yesterday some one  create some users if we want add tomorrow that person not available if we add today
1.first find secrete from oneshift-config,as well as  oc get oauth/cluster will get name
2.oc extract secret/htpasswd-secet --to=. (to download on your local miachine,- means display on screen)
3.first down load and delete old secrete or update on with same secret name if we delete or update again pod will recrete on openshift-authentication

how to delete
=====
download secrete using extract,htpasswd -D htpasswd-file user5(its delete on local level if want cluster level)
oc set data secret/name --from-file=htppaswd=htpasswd-file -n openshift-config --for update secrete file 
oc extract secret/htpassword-secret --to=- 
oc get user
oc delete user user5 -->it shw bcz of catch memory
oc get identity
oc delete identity user5
oc login -u ues5 -p redhat
htpasswd -b -B htpasswd user4 red --for change password
oc set data secret/name --from-file=htppaswd -n openshift-config

RBAC--role based access control
Rule--> Allowed actions for objects or groups of objects.
Role--> Sets of rules. Users and groups can be associated with multiple roles.--cluster role.role,cluster role binding,role binding
Binding--> Assignment of users or groups to a role.
role will assign to binding ,binding will assign to user


self-provisioner Users with this role can create projects. It is a cluster role, not a
project role.--every one will get this acces by default(

oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth --->we have to delete so that no one will access except user2 see below command

oc adm policy add-role-to-user view user2 -n swami

oc get clusterrole
oc desribe clusterrole cluster-status
two types of role cluster and local(name space level)=================================

oc adm policy add-cluser-role-to-user cluster-admin user1
oc adm policy remove-cluster-role-from-user cluster-admin user1 
oc adm policy who-can delete  user1  -to see who can delete

oc auth can-i delete no --as user1

oc adm policy add-role-to-user admin user1 -n namespace
oc adm policy add-role-to-user view user2 -n namespace
ks get users

oc adm groups new cluster-admin
oc adm groups new sap-ns
oc get groups
oc adm groups add-users sap-ns user1 user2
oc adm policy add-cluser-role-to-group cluster-admin SAp-ns

oc adm groups remove-users sap-ns user1


=============================================
network
==========================
dry-run=client will show basic configuration and check the syntax but dry-run=server will show lot of extra timestamp,and some extra lines so it will get some conflict while create app

debug:
==========
oc debg will work to login if node have healthy(in ready state,otherwise we have to login with ssh -l core but require ssh keys)
1.fist new temporary name space will create inside that pod also will create from that you can login to node if you want access binaries then we have to run chroot /host the we can access binaries once activity is done ,if we logout will auto destroy pod and name space 

b/g deployment--from v1 to v2 change svc on router,and revert to v2 for that change svc on router
canary-ab-deployment--80% will go to v1,20% will go to v2,if v2 looks good day by day they will increase trafic

canary-ab-deployment
====================
in route

to:
    kind: Service
    name: service-a
    weight: 80
  alternateBackends:
  - kind: Service
    name: service-b
    weight: 20
  port:
    targetPort: 80

route two types
1.unsecure route --http
2.security routes--https
security routes
----------
edge--validate cert and key at route level then request go as http to application
re-encryption-- validate cert,key  at route level aswell as application end
passthrough-- validate cert ,key at  application end 

user-->route-->application
edge-->https-->http
passtrough-->http-->https
re-encrypt-->https-->https


we have to create cert and highlevel need to signed then only it will validate(cert,key will create with openssl)

network policy(netpol)
======================
normaly all pods will communicate each other
arund 15 network policy will be there but main netpol is


1.deny-all 
2.allow from same namespace(if two pods have on same namespace if they want communicate on each other)
3.allow from ingress( route to access outside)
4.allow-for-monitor (logs,metrics)


different name space-->spec:from:namespaceselector:-matchlabels:allow:newtwork1(oc label ns/network-3 allow:network1)if you want give only one pod from network1(padselector:matchlable:deployment:app02)

svc--
=========.
how to access app outside
================
if we want http(80),https(443)  application we can access with route(-->service--clusterip-->route)
clusterip is bydefault
if we want access application(db)  we can use nodeport,loadbalancer and it support  http(80),https(443) also
nodeport-->we can access with node ip and port but security wise not good as well as if node down  then we cant access so we will use for testing purpose
loadbalancer-->it will get ip from metallb its need to configure, (cloud also have loadbalancer like aws its costly)
externalname
multus seconday interface--> ifwe want configure we have to do nmstate,nnc,nncp,nad configuraton this one 1%  onley will use

=====================

qota and resource quota both are same
resource quota--hard means upto we can create pod (limit and request will be there in deploy)-->set the quota on name space level
limit range -->pod level to control(min ,max) -->application team know how much space consume we dont know so this not importent
project template
======================

Configure default quotas, limit ranges, role bindings, and other restrictions for new projects, and
the allowed users to self-provision new projects.

Replace the namespace name with the ${PROJECT_NAME} value.
oc adm creat-bootstrap-project-template -oyml >project-template.yml
add resource quota and network policy 
and apply on openshift-config -->its global config


oc edit projects.config.openshift.io cluster --->below lines need to update

spec:
projectRequestTemplate:
name: project-request

then watch openshift-apiserver  -->pads recreating on apiserver name space



cluster operator
=============

Cluster Operator is a piece of software that automates the management of a specific application or service within the cluster. Cluster Operators are designed to handle both the initial setup (Day 1 operations) and ongoing management (Day 2 operations) of applications2.
weburi-->operator-->1.operatorhub 2.installed operators
from operatorhub we will install softwaresas per reauirement  like virtualization,falcon
source
=========
Red Hat
Red Hat packages, ships, and supports operators in this catalog.--get support from redhat
Certified
Independent software vendors support operators in this catalog. -->redhat wont support if we have issue ,who are proving that company help
Community
Operators without official support.--redhat wont support if we have issue ,who are proving that company help

Marketplace
Commercial operators that you can buy from Red Hat Marketplace.

application team need more  feature all team will go trought helm
most common operator will use  compliance operator its for scan cluster(its advise create netpol on some namespace if not good as secure wise)

if any one ask what are operator your used web terminal(we can login as cli from os),login,monitoring,compliance operator


sarch operator-->select operator-->frst is ask channel(fast,stable),version, on which name space need to instal(bydefault all name space)-->update approval-->manual,auto-->default
to check status
installed operators

cli
==
operater group and subscription yml file(oc get og -oyml,sub)
to check status oc get csv



