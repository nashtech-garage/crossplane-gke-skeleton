# Crossplane to create a GKE cluster
## Install the official GCP Container Provider
```bash
# gke_provider.yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-gcp-container
spec:
  package: xpkg.upbound.io/upbound/provider-gcp-container:v1.8.0
```
```bash
kubectl apply -f gke_provider.yaml
```
## Create a Kubernetes secret
The provider-gcp-container requires credentials to create and manage GCP resources.
Generate a GCP JSON key file Create a JSON key file containing the GCP account credentials. GCP provides documentation on how to create a key file.

Here is an example key file:
```json
{
  "type": "service_account",
  "project_id": "caramel-goat-354919",
  "private_key_id": "e97e40a4a27661f12345678f4bd92139324dbf46",
  "private_key": "fdfjf",
  "client_email": "my-sa-313@caramel-goat-354919.iam.gserviceaccount.com",
  "client_id": "103735491955093092925",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/my-sa-313%40caramel-goat-354919.iam.gserviceaccount.com"
}
```
Save this JSON file as gcp-credentials.json.
## Create a Kubernetes secret with GCP credentials
Use kubectl create secret -n upbound-system to generate the Kubernetes secret object inside the Universal Crossplane cluster.
```bash
kubectl create secret generic gcp-secret -n upbound-system --from-file=creds=./gcp-credentials.json
```
## Create a ProviderConfig
Create a ProviderConfig Kubernetes configuration file to attach the GCP credentials to the installed official provider-gcp-container.
```bash
apiVersion: gcp.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  projectID: crossplaneprojects
  credentials:
    source: Secret
    secretRef:
      namespace: upbound-system
      name: gcp-secret
      key: creds
```