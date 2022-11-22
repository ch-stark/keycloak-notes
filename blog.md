## ACM/ACS SSO Integration

Authors:

Goal of this blog:

Customers should have a seamless login experience for a user that has OpenShift Platform Plus (OPP) to authenticate to OpenShift and services such as RHACS and RHACM that come along with it. 
The main motivation for  comes from the actual pain of configuring SSO for a fleet of clusters managed by RCACM. 
We would like to demonstrate how to deploy Keycloak and configure it to manage a fleet of clusters and OPP Services such as ACS.

# Architecture

We have one Hub-Cluster. On this Cluster Keycloak will be setup and also ACS central has been installed. There is another ManagedCluster

High level Requirements

User should be able to login to the OpenShift console, and to ACM console as well as ACS console with the same identity 
Federate to external/Enterprise IDPs using Keycloak  
Group information from IDP should be synchronized with the cluster. Keycloak which pulls in group information in the token should be able to apply this across clusters similar to how ldap group sync applies to a single cluster.  



## Configure and Install Keycloak operator on the hub cluster (not fips compliant)

Please note that a new Operator will be rewritten from scratch to provide the best experience for the Quarkus distribution. While the legacy Operator is now deprecated and will reach EOL with Keycloak 20, the new one is already available as a preview, see the installation guide.
One of the most common concerns around the new Operator is the current lack of the CRDs for managing Keycloak resources, such as realm, users and clients, in a cloud-native way. One of the key aspects of the new Operator will be redesign of managing these Keycloak resources via CRs and git-ops. This new approach will leverage the new storage architecture and future immutability options, making the CRs the declarative single source of truth. In comparison to the legacy Operator, this will bring high robustness, reliability, and predictability to the whole solution.


Procedure:

- Sign into Keycloak with admin credentials 
- Setup Realm for ACM under which all the configurations for the fleet of clusters will reside. Typically each realm is for an isolated set of Users, groups, Clients and IDP associated with them. 
- Configure Identity Provider in Keycloak. Keycloak will redirect users immediately to auth.redhat.com which acts as the Authorization OIDC server. 

- Authentication Flow configured is Basic browser based flow with redirect to the configured IDP to authenticate incase cookies are deleted or time out

- Clients: Client config is probably the most important aspect of this workflow. There is a client created for each Spoke/Managed cluster and one associated with each Service such as ACS 
  spoke/managed clusters 
- Spoke/Managed clusters have OAuth servers configured to use OIDC with Issuer url pointing to the Keycloak instance on the Hub cluster
- Role Bindings need to be created to map the groups to cluster roles.  


### Configure SSO for ACS 


with ACS you can have multiple auth providers.. including OpenShift
you can consume the OCP one and use what you already have.. or use another openid as you like
for example  
you can configure OCP to use keycloak and configure ACS to use OCP oauth! fine.!!
another option is to configure ACS to consume keycloak as auth provider, you will get the same result!


### Possible Extension of the solution

From ACM-Point of view we can create a SSO-PolicySet

on the hub configure Keycloak
on the spoke setup OAuth and all the necessary secrets and configmaps















