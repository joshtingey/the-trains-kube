# thetrainskube

This is my own personal version of [hobby-kube](https://github.com/hobby-kube) for deploying a kubernetes cluster on digitaloceon.

## Setup

The following packages are required to be installed locally:

```
$ sudo apt install kubectl jq wireguard-tools
```

You also need to install the latest version of [terraform](https://www.terraform.io/downloads.html) and put the unzipped executable in your PATH.

Modules are using ssh-agent for remote operations. Add your SSH key with `ssh-add -K` if Terraform repeatedly fails to connect to remote hosts.

### Terraform Configuration

Export the following environment variables depending on the modules you're using:

```
$ export TF_VAR_domain=<domain> # e.g. example.org
```

```
$ export TF_VAR_digitalocean_token=<token>
```

```
$ export TF_VAR_digitalocean_ssh_keys='["<key-id>"]'
```

You can get SSH key IDs using [this API](https://developers.digitalocean.com/documentation/v2/#list-all-keys).

### Using Terraform to create cluster

From the root of this project...

fetch the required modules

```
$ terraform init
```

see what `terraform apply` will do

```
$ terraform plan
```

execute it

```
$ terraform apply
```

destroy cluster

```
$ terraform destroy
```

### Applying Manifests

Create the nginx ingress

```
$ kubectl apply -f manifests/ingress
```

Setup TLS (first change the email address in cert-manager.yml to yours)

```
$ kubectl apply -f manifests/ingress/tls
```

For gitlab integration

```
$ kubectl apply -f manifests/gitlab-admin-service-account.yaml
```

Setup for rook-ceph storage

```
$ kubectl apply -f manifests/storage/common.yaml
$ kubectl apply -f manifests/storage/operator.yaml # Make sure to wait till this is fully deployed
$ kubectl apply -f manifests/storage/cluster.yaml
```
To use rook-ceph toolbox

```
$ kubectl apply -f manifests/storage/toolbox.yaml
```
