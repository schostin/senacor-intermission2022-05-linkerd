# External DNS

I am using the external-dns chart of bitnami and I fill it with pre-defined values.

## Workload Identity GKE

I am using workload identity in order to authenticate external-dns to the dns managed zone.

In order to so the following was needed upfront:

- Create a service account for external dns in the project where the cluster lives with
  `gcloud iam service-accounts create k8s-external-dns`
- create a kubernetes service account in the namespace with
  `kubectl create serviceaccount external-dns --namespace external-dns`
- Grant the role DNS admin or the project (or on the DNS zone) with

  ```bash
  gcloud projects add-iam-policy-binding "$GOOGLE_PROJECT_ID" \
    --member "serviceAccount:k8s-external-dns@"$GOOGLE_PROJECT_ID".iam.gserviceaccount.com" \
    --role "roles/dns.admin"
  ```

- Grant the K8s service account the right to impersonate the Google service account with

  ```bash
  gcloud iam service-accounts add-iam-policy-binding k8s-external-dns@"$GOOGLE_PROJECT_ID".iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:"$GOOGLE_PROJECT_ID".svc.id.goog[external-dns/external-dns]"
  ```

- Annotate the k8s service account with the google service account with

  ```bash
  kubectl annotate serviceaccount external-dns \
    --namespace external-dns \
    iam.gke.io/gcp-service-account=k8s-external-dns@"$GOOGLE_PROJECT_ID".iam.gserviceaccount.com
  ```

## Installation

I created a namespace manually by running `kubectl create ns external-dns`.
Afterwards configure helm and add the chart repository via
`helm repo add bitnami https://charts.bitnami.com/bitnami`.

The installation can then be done with
`helm upgrade --install external-dns bitnami/external-dns --namespace external-dns --values ./values.yaml`
