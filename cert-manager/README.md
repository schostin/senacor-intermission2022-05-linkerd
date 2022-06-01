# Cert Manager

I am using the cert-manager chart of jetstack and I fill it with pre-defined values.

## Workload Identity GKE

I created a namespace manually by running `kubectl create ns cert-manager`.

I am using workload identity in order to authenticate cert-manager to the dns managed zone for
dns challenges.

In order to so the following was needed upfront:

- Create a service account for cert-manager in the project where the cluster lives with
  `gcloud iam service-accounts create k8s-cert-manager`
- create a kubernetes service account in the namespace with
  `kubectl create serviceaccount cert-manager --namespace cert-manager`
- Grant the role DNS admin or the project (or on the DNS zone) with

  ```bash
  gcloud projects add-iam-policy-binding "$GOOGLE_PROJECT_ID" \
    --member "serviceAccount:k8s-cert-manager@"$GOOGLE_PROJECT_ID".iam.gserviceaccount.com" \
    --role "roles/dns.admin"
  ```

- Grant the K8s service account the right to impersonate the Google service account with

  ```bash
  gcloud iam service-accounts add-iam-policy-binding k8s-cert-manager@"$GOOGLE_PROJECT_ID".iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:"$GOOGLE_PROJECT_ID".svc.id.goog[cert-manager/cert-manager]"
  ```

- Annotate the k8s service account with the google service account with

  ```bash
  kubectl annotate serviceaccount cert-manager \
    --namespace cert-manager \
    iam.gke.io/gcp-service-account=k8s-cert-manager@"$GOOGLE_PROJECT_ID".iam.gserviceaccount.com
  ```

## Installation

I created a namespace manually by running `kubectl create ns cert-manager`.
Afterwards configure helm and add the chart repository via
`helm repo add jetstack https://charts.jetstack.io`.

The installation can then be done with
`helm upgrade --install cert-manager jetstack/cert-manager --namespace cert-manager --values ./values.yaml`

To have clusterIssuer I manually applied the [staging-clusterissuer](./staging-clusterissuer.yaml) for tryouts
and the [clusterissuer](./clusterissuer.yaml) for real production use-cases after tryouts succeeded. This
can the be tested by applying the [test-certificate](./test-certificate.yaml).
