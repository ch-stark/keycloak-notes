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


```
# get spoke cluster's redirect url
echo "$(oc get -n openshift-authentication cm v4-0-config-system-metadata -o jsonpath='{.data.oauthMetadata}' | jq -r '.issuer')/oauth2callback/keycloak"

# get route's ca
oc -n openshift-ingress-operator get secret router-ca -o jsonpath="{ .data.tls\.crt }" | base64 -d -i > ca.crt
oc -n openshift-config create cm keycloak-ca --from-file=ca.crt

# create client secret
oc -n openshift-config create secret generic keycloak-idp-secret --from-literal=clientSecret=<secret>

# check current idps status
oc describe oauth cluster

# create keycloak idp
oc apply -f ocp-idp.yaml

# create binding between cluster-admin group and role
oc apply -f cluster-admin-from-group.yaml

# remove default binding for unauthorized users
oc describe clusterrolebindings basic-users
oc patch clusterrolebindings basic-users --type json -p='[{"op": "remove", "path": "/subjects/0"}]'

```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-from-group
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: cluster-admin
```


Configure OAuth on the Spoke/ManagedCluster

```
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: keycloak
    mappingMethod: claim
    type: OpenID
    openID:
      clientID: spoke-cluster
      clientSecret:
        name: keycloak-idp-secret
      ca: 
        name: keycloak-ca
      claims:
        preferredUsername: 
        - preferred_username
        name: 
        - given_name
        - name
        email: 
        - email
        groups: 
        - ocp-groups
      issuer: https://keycloak.apps.cluster0.aws.ocp.team/auth/realms/acm # needs to match current KC address
```

### Configure SSO for ACS 


with RHACS you can have multiple auth providers.. including OpenShift
you can consume the OCP one and use what you already have.. or use another openid as you like
for example  
you can configure OCP to use keycloak and configure ACS to use OCP oauth! fine.!!
another option is to configure ACS to consume keycloak as auth provider, you will get the same result!


### Possible Extension of the solution

From ACM-Point of view we can create a SSO-PolicySet

on the hub configure Keycloak
on the spoke setup OAuth and all the necessary secrets and configmaps






Special thanks goes to









