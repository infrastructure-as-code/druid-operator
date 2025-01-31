## Install the operator

- Register the `Druid` custom resource definition (CRD).

```
# Setup Service Account
druid-operator$ kubectl create -f deploy/service_account.yaml
# Setup RBAC
druid-operator$ kubectl create -f deploy/role.yaml
druid-operator$ kubectl create -f deploy/role_binding.yaml
# Setup the CRD
druid-operator$ kubectl create -f deploy/crds/druid.apache.org_druids_crd.yaml

# Update the operator manifest to use the druid-operator image name (if you are performing these steps on OSX, see note below)
druid-operator$ sed -i 's|REPLACE_IMAGE|<druid-operator-image>|g' deploy/operator.yaml
# On OSX use:
druid-operator$ sed -i "" 's|REPLACE_IMAGE|<druid-operator-image>|g' deploy/operator.yaml

# Deploy the druid-operator
druid-operator$ kubectl create -f deploy/operator.yaml

# Check the deployed druid-operator
druid-operator$ kubectl describe deployment druid-operator
```
- Operator can be deployed with namespaced scope or clutser scope. By default the operator is namespaced scope.
- For the operator to be cluster scope, the following params inside the ```operator.yaml``` need to be changed.
```
- name: WATCH_NAMESPACE
  value: ""
```
- Use ```ClusterRole``` and ```CluterRoleBinding``` instead of ```role```and ```roleBinding```.


## Deny List in Operator
- There may be use cases where we want the operator to watch all namespaces but restrict few namespaces, due to security, testing flexibility etc reasons.
- The druid operator supports such cases. In ```deploy/operator.yaml```, user can enable ```DENY_LIST``` env and pass the namespaces to be excluded. Each namespace to be seperated using a comma.

## Reconcile Time in Operator
- As per operator pattern, the druid operator reconciles every 10s ( default reconcile time ) to make sure the desired state ( druid CR ) in sync with current state.
- In case user wants to adjust the reconcile time, it can be adjusted by adding an ENV variable in ```deploy/operatoryaml```, user can enable ```RECONCILE_WAIT``` env and pass in the value suffixed with ```s``` string ( example: 30s). The default time is 10s.

## Deploy a sample Druid cluster

- An example spec to deploy a tiny druid cluster is included. For full details on spec please see `pkg/api/druid/v1alpha1/druid_types.go`

```
# deploy single node zookeeper
druid-operator$ kubectl apply -f examples/tiny-cluster-zk.yaml

# deploy druid cluster spec
druid-operator$ kubectl apply -f examples/tiny-cluster.yaml
```

## Debugging Problems

 - For kubernetes version 1.11 make sure to disable ```type: object``` in the CRD root spec. 

```
# get druid-operator pod name
druid-operator$ kubectl get po | grep druid-operator

# check druid-operator pod logs
druid-operator$ kubectl logs <druid-operator pod name>

# check the druid spec
druid-operator$ kubectl describe druids tiny-cluster

#  check if druid cluster is deployed
druid-operator$ kubectl get svc | grep tiny
druid-operator$ kubectl get cm | grep tiny
druid-operator$ kubectl get sts | grep tiny
```
