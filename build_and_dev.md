
# No Chart.lock not already built
helm dependency build .

helm dependency update

# Render chart locally

helm template . --generate-name

helm install --generate-name --dry-run --debug qa-report-platform


# Test it out ... 

$namespace='g0-dev2'
$release='qarp1'


<!-- export TESTNS=sl3-mci
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: ${TESTNS}
EOF -->

helm install $release . -n $namespace --create-namespace -f values.yaml

helm upgrade $release . -n $namespace -f values.yaml

## Some useful commands

```sh

# Connect to container example
kubectl exec -i -t -n ${TESTNS} consul-server-0 -c consul "--" sh -c "clear; sh"

kubectl port-forward svc/consul-ui -n ${TESTNS} 8080:80
kubectl port-forward svc/sl3-fabric-vault-ui -n ${TESTNS} 8200:8200

```
