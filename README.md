# Keycloak

This repository contains the tooling for deploying Keycloak application into running Kubernetes cluster. It is using PostgreSQL for persisitence.

## Prerequisites

* Kubernetes cluster with Nginx based ingress controller
* OpenSSL (for generating SSL certificates)
* Amazon AWS access for Route53 records
* Ansible 2.2
* kubectl with proper configuration to connect to running Kubernetes cluster
* boto library which will be used for communication with Amazon AWS APIs

## Configuration

The configuration is in `group_vars/all/vars.yaml`. It configures different details of the deployment.

| Variable | Explanation | Example |
|--------|-------------|---------|
| `namespace` | Kubernetes namespace where Keycloak should be deployed | `dave` |
| `keycloak_release` | Which Docker image tag should be used in the deployment of the Keycloak application | `3.2.0.Final` |
| `dns_zone` | Hosted DNS zone which has to exist in Route53 | `dbg-devops.com` |
| `keycloak_dns` | Hostname of the UI | `snapshot.dave.dbg-devops.com` |
| `elb_hosted_zone` | The hosted zone in which the aliased ELB load balancers are hosted (should be dependent on the AWS region) | `Z32O12XQLNTSW2` |
| `key_path` | Path to the SSL private key | `./api.key` |
| `cert_path` | Path to the SSL public key | `./api.cert` |
| `admin_user` | Admin user for Keycloak service | `admin` |
| `admin_password` | Admin password for Keycloak service | `123456` |
| `postgres_user` | Admin user for Keycloak service | `keycloak` |
| `postgres_password` | Admin password for Keycloak service | `123456` |
| `postgres_database` | NAme of the postgreSQL database to use | `keycloak` |

Following configuration is needed only for signing the certificates with Let's Encrypt:

| Variable | Explanation | Example |
|--------|-------------|---------|
| `letsencrypt_account_key_path` | Path where the Let's Encrypt account key is / should be created | `./account.key` |
| `acme_directory` | Let's Encrypt ACME directory where we the certificates should be signed (production or staging) | `https://acme-staging.api.letsencrypt.org/directory` |
| `full_chain_cert` | URL to chain of public CA certificates which should be used with the end certificates. If not specified, only the end certificate will be used. | `https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt` |

## Installation

To install the application export the variables with AWS access tokens and run:
```
ansible-playbook install.yaml
```

It will create the Kubernetes resources, Route53 records etc. The definition of the Kubernetes objects is in the templates of different roles. Change the templates if you want to change something. **Before running the installation, check the configuration first.**

## Uninstallation

To uninstall the application export the variables with AWS access tokens and run:
```
ansible-playbook uninstall.yaml
```

It will remove all Kubernetes resources from the cluster as well as the Route53 DNS records. **Before running the uninstallation, check the configuration first.**

## Signed certificates with Let's Encrypt

Playbook `letsencrypt.yaml` can be used to obtain signed keys from Let's Encrypt CA. Export the variables with AWS access tokens and run:
```
ansible-playbook letsencrypt.yaml
```

* If the account key is missing, it will generate new account key
* If the key is missing, it will generate new key with the hostname in CN
* It will generate CSR
* It will try to get the certificates signed. DNS verification will be used - Route53 will be automatically used to handle it.
* The keys will be signed only if the old keys expire in 10 or less days

*The `acme_directory` variable can be used to define whether the production Let's Encrypt service should be used or the staging service (for testing).*
