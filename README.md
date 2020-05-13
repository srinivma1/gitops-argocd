# gitops-argocd

NSTALLING ARGOCD ON OPENSHIFT
In this blog series we are using ArgoCD as it provides a great CLI and WebUI, future blogs will potentially explore different alternatives like Flux.

In order to deploy ArgoCD on OpenShift 4 you can go ahead and follow the following steps as a cluster admin:

```
Deploy ArgoCD components on OpenShift

# Create a new namespace for ArgoCD components

oc create namespace argocd

# Apply the ArgoCD Install Manifest

oc -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.2.2/manifests/install.yaml

# Get the ArgoCD Server password

ARGOCD_SERVER_PASSWORD=$(oc -n argocd get pod -l "app.kubernetes.io/name=argocd-server" -o jsonpath='{.items[*].metadata.name}')

Patch ArgoCD Server Deployment so we can expose it using an OpenShift Route
# Patch ArgoCD Server so no TLS is configured on the server (--insecure)

PATCH='{"spec":{"template":{"spec":{"$setElementOrder/containers":[{"name":"argocd-server"}],"containers":[{"command":["argocd-server","--insecure","--staticassets","/shared/app"],"name":"argocd-server"}]}}}}'

oc -n argocd patch deployment argocd-server -p $PATCH

# Expose the ArgoCD Server using an Edge OpenShift Route so TLS is used for incoming connections

oc -n argocd create route edge argocd-server --service=argocd-server --port=http --insecure-policy=Redirect

Deploy ArgoCD Cli Tool
# Download the argocd binary, place it under /usr/local/bin and give it execution permissions

curl -L https://github.com/argoproj/argo-cd/releases/download/v1.2.2/argocd-linux-amd64 -o /usr/local/bin/argocd

chmod +x /usr/local/bin/argocd

Update ArgoCD Server Admin Password
# Get ArgoCD Server Route Hostname

ARGOCD_ROUTE=$(oc -n argocd get route argocd-server -o jsonpath='{.spec.host}')

# Login with the current admin password

argocd --insecure --grpc-web login ${ARGOCD_ROUTE}:443 --username admin --password ${ARGOCD_SERVER_PASSWORD}

# Update admin's password

argocd --insecure --grpc-web --server ${ARGOCD_ROUTE}:443 account update-password --current-password ${ARGOCD_SERVER_PASSWORD} --new-password

```

Now you should be able to use the ArgoCD WebUI and the ArgoCD Cli tool to interact with the ArgoCD Server
