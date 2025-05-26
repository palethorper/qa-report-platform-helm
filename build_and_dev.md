
# No Chart.lock not already built
helm dependency build .

helm dependency update

# Render chart locally

helm template . --generate-name

helm install . --generate-name --dry-run --debug 


# Test it out ... 
```sh

source ./.env.g0-auth

source ./.env.g0-dev1

source ./.env.devlocal




kubectl create namespace ${NAMESPACE}

# DO container registry secret
kubectl apply -f ../../secrets/g0-cr-secret.yaml -n ${NAMESPACE}


#################################################################33
# Local (self-hosted or preminted certs only
# SSL cert - slf signed for local
# Create this autopmatically for non-locals
export CERT_PATH="../../secrets/local-k8s"
kubectl create secret tls -n ${NAMESPACE} tls-certificate --key ${CERT_PATH}/cert.key --cert ${CERT_PATH}/cert.crt
#################################################################3

# kubectl apply -n ${NAMESPACE} -f "../../secrets/azure-blob-storage-secret.yaml"

## Secrets

kubectl create secret generic -n ${NAMESPACE} platform-creds --from-literal=opensearch-admin-password=${OS_PASSWORD} --from-literal=OS_PASSWORD=${OS_PASSWORD}

kubectl create secret generic -n ${NAMESPACE} message-broker-secret --from-literal=client-passwords=${MB_PASSWORD} --from-literal=inter-broker-password=${MB_PASSWORD} --from-literal=inter-broker-client-secret=${MBPASS} --from-literal=controller-password=${MB_PASSWORD} --from-literal=controller-client-secret=${MB_PASSWORD}

# BUG: Digitial ocean \ k8s load balaner
# https://github.com/digitalocean/digitalocean-cloud-controller-manager/blob/master/docs/controllers/services/examples/README.md#accessing-pods-over-a-managed-load-balancer-from-inside-the-cluster
helm upgrade \
    --create-namespace \
    --install ${RELEASE} ${INSTALL_FROM} \
    --namespace ${NAMESPACE} \
    -f ${VALUES} \
    --set global.ospassword=${OS_PASSWORD} \
    --set global.hostname=${QAIENV_HOSTNAME} \
    $@

helm uninstall ${RELEASE} -n ${NAMESPACE}

# Install 
# helm install \
#     --create-namespace \
#     ${RELEASE} ${INSTALL_FROM} \
#     -n ${NAMESPACE} \
#     -f ${VALUES} \
#     --set global.ospassword=${OS_PASSWORD} \
#     --set global.hostname=${QAIENV_HOSTNAME} \
#     $@

# #######################

# ```

# ```powershell
# $namespace='g0-auth-enabled'
# $release='qarp-2'
# $ospassword='' # Sample password for testing - do not use this  for prod deploys 

# kubectl create namespace $namespace
# kubectl create secret generic -n $namespace platform-creds --from-literal=opensearch-admin-password=$ospassword

# helm install $release . -n $namespace --create-namespace -f values.yaml

# helm upgrade --install --create-namespace $release . -n $namespace -f values.yaml

# helm uninstall $release -n $namespace 

# curl -k -vvv https://localhost:9200/ -u 'admin:Test@Password01'

# ```

# ## Some useful commands

# ```sh

# # Connect to container example
# kubectl exec -i -t -n ${TESTNS} consul-server-0 -c consul "--" sh -c "clear; sh"

# kubectl port-forward svc/consul-ui -n ${TESTNS} 8080:80
# kubectl port-forward svc/sl3-fabric-vault-ui -n ${TESTNS} 8200:8200

# ```
