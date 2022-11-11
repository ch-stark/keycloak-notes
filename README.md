# keycloak-notes


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
