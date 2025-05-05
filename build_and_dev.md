
# No Chart.lock not already built
helm dependency build .

helm dependency update

# Render chart locally

helm template . --generate-name

helm install --generate-name --dry-run --debug qa-report-platform


# Test it out ... 

```powershell
$namespace='g0-auth-enabled'
$release='qarp-2'
$ospassword='Test@Password01' # Sample password for testing - do not use this  for prod deploys 

kubectl create namespace $namespace
kubectl create secret generic -n $namespace platform-creds --from-literal=opensearch-admin-password=$ospassword

helm install $release . -n $namespace --create-namespace -f values.yaml

helm upgrade --install --create-namespace $release . -n $namespace -f values.yaml

helm uninstall $release -n $namespace 



curl -k -vvv https://localhost:9200/ -u 'admin:Test@Password01'

```

## Some useful commands

```sh

# Connect to container example
kubectl exec -i -t -n ${TESTNS} consul-server-0 -c consul "--" sh -c "clear; sh"

kubectl port-forward svc/consul-ui -n ${TESTNS} 8080:80
kubectl port-forward svc/sl3-fabric-vault-ui -n ${TESTNS} 8200:8200

```
