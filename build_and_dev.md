
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

kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -

# DO container registry secret
kubectl apply -f ../../secrets/g0-cr-secret.yaml -n ${NAMESPACE}


#################################################################33
# Local (self-hosted or preminted certs only
# SSL cert - slf signed for local
# Create this autopmatically for non-locals
export CERT_PATH="../../secrets/self-signed-devlocal"
kubectl create secret tls -n ${NAMESPACE} tls-certificate --key ${CERT_PATH}/cert.key --cert ${CERT_PATH}/cert.crt  

kubectl create secret generic ca-secret -n ${NAMESPACE} --from-file=tls.crt=${CERT_PATH}/server.crt --from-file=tls.key=${CERT_PATH}/server.key --from-file=ca.crt=${CERT_PATH}/ca-cert.pem

#################################################################3

# kubectl apply -n ${NAMESPACE} -f "../../secrets/azure-blob-storage-secret.yaml"

## Secrets

kubectl create secret generic -n ${NAMESPACE} platform-creds --from-literal=OS_PASSWORD=${OS_PASSWORD} --from-literal=SEND_DATA_USER=${SEND_DATA_USER} --from-literal=SEND_DATA_PASSWORD=${SEND_DATA_PASSWORD}

# kubectl create secret generic -n ${NAMESPACE} message-broker-secret --from-literal=client-passwords=${MB_PASSWORD} --from-literal=inter-broker-password=${MB_PASSWORD} --from-literal=inter-broker-client-secret=${MBPASS} --from-literal=controller-password=${MB_PASSWORD} --from-literal=controller-client-secret=${MB_PASSWORD}


# kubectl apply -f https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.46.0/strimzi-crds-0.46.0.yaml -n ${NAMESPACE}

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
```


----
## Notes

https://medium.com/@yago82/secure-communication-setup-ssl-certificates-for-logstash-and-filebeat-with-openssl-bc1c7827ee3b

----

```sh
# Post to https endpoint (full file)
curl -k -vvv https://devlocal.groundzero.solutions:50000 -X POST -H "Content-Type: application/xml"  --cacert ../../secrets/self-signed-devlocal/ca-cert.pem -H "Tags: nunit,selenium" --data @nunit3-results.xml -u "${SEND_DATA_USER}:${SEND_DATA_PASSWORD}"


openssl s_client -showcerts -connect ${QAIENV_HOSTNAME}:50000 -servername ${QAIENV_HOSTNAME} 2>/dev/null | openssl x509 -text

```
