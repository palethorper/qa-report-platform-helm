
# No Chart.lock not already built
helm dependency build .

helm dependency update

# Render chart locally

helm template . --generate-name

helm install --generate-name --dry-run --debug qa-report-platform


# Test it out ... 
```sh

export VALUES="${VALUES:-values.yaml}"
export RELEASE="${RELEASE:-qarp-2}"
export NAMESPACE="${NAMESPACE:-g0-auth-enabled}"
export INSTALL_FROM="${INSTALL_FROM:-.}"

export QAIENV_HOSTNAME='g0-auth.groundzero.solutions'

export OS_PASSWORD="${OS_PASSWORD:-Test@Password01}"

export MBPASS='P@ssw0rd11!!'

kubectl create secret generic -n ${NAMESPACE} platform-creds --from-literal=opensearch-admin-password=${OS_PASSWORD} --from-literal=OS_PASSWORD=${OS_PASSWORD}

kubectl create secret generic -n ${NAMESPACE} message-broker-secret --from-literal=client-passwords=${MBPASS} --from-literal=inter-broker-password=${MBPASS} --from-literal=inter-broker-client-secret=${MBPASS} --from-literal=controller-password=${MBPASS} --from-literal=controller-client-secret=${MBPASS}


helm install \
    --create-namespace \
    ${RELEASE} ${INSTALL_FROM} \
    --namespace ${NAMESPACE} \
    -f ${VALUES} \
    $@

helm upgrade \
    --create-namespace \
    --install ${RELEASE} ${INSTALL_FROM} \
    --namespace ${NAMESPACE} \
    -f ${VALUES} \
    $@

#######################
# DEv local settings
#######################

export VALUES="${VALUES:-values.local.yaml}"
export RELEASE="${RELEASE:-qai1}"
export NAMESPACE="${NAMESPACE:-devlocal}"
export INSTALL_FROM="${INSTALL_FROM:-.}"

export MB_PASSWORD='Gr0un40Test!!'
export OS_PASSWORD='Gr0un40Test!!'

export QAIENV_HOSTNAME='devlocal.groundzero.solutions'

kubectl create namespace ${NAMESPACE}

# DO container registry secret
kubectl apply -f ../../secrets/g0-cr-secret.yaml -n ${NAMESPACE}

# SSL cert - slf signed for local
export CERT_PATH="../../secrets/local-k8s"
kubectl create secret tls -n ${NAMESPACE} tls-certificate --key ${CERT_PATH}/cert.key --cert ${CERT_PATH}/cert.crt

## Secrets
kubectl create secret generic -n ${NAMESPACE} platform-creds --from-literal=opensearch-admin-password=${OS_PASSWORD} --from-literal=OS_PASSWORD=${OS_PASSWORD}

kubectl create secret generic -n ${NAMESPACE} message-broker-secret --from-literal=client-passwords=${MB_PASSWORD} --from-literal=inter-broker-password=${MB_PASSWORD} --from-literal=inter-broker-client-secret=${MBPASS} --from-literal=controller-password=${MB_PASSWORD} --from-literal=controller-client-secret=${MB_PASSWORD}


# helm install ${RELEASE} ${INSTALL_FROM} -n ${NAMESPACE} --create-namespace -f ${VALUES}

helm install \
    --create-namespace \
    ${RELEASE} ${INSTALL_FROM} \
    -n ${NAMESPACE} \
    -f ${VALUES} \
    --set global.ospassword=${OS_PASSWORD} \
    --set global.hostname=${QAIENV_HOSTNAME} \
    $@

helm upgrade \
    --create-namespace \
    --install ${RELEASE} ${INSTALL_FROM} \
    --namespace ${NAMESPACE} \
    -f ${VALUES} \
    --set global.ospassword=${OS_PASSWORD} \
    --set global.hostname=${QAIENV_HOSTNAME} \
    $@

helm uninstall ${RELEASE} -n ${NAMESPACE}

#######################

```

```powershell
$namespace='g0-auth-enabled'
$release='qarp-2'
$ospassword='' # Sample password for testing - do not use this  for prod deploys 

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
