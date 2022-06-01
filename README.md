# Senacor Intermission 05-2022

LinkerD tryouts for the intermission 2022.

This repository contains documentation and code samples of the deployed resources to the GKE cluster.

## Pre-requisites

You'll some **Kubernetes cluster** running and best to also have a **DNS zone** somewhere. I used external-dns
in order to create DNS entries automatically in my DNS Managed Zone that I created. I also used
cert-manager to get certificates for the e.g. API gateway to get Let's encrypt certificates.

### Components

#### External-DNS

The helm templates and further documentation on how to install this can be access in the folder [external-dns](./external-dns/).

#### Cert-Manager

The helm templates and further documentation on how to install this can be access in the folder [cert-manager](./cert-manager/).

## LinkerD installation and tryouts

All of that documentation can be accessed in the [linkerd](./linkerd) folder.
