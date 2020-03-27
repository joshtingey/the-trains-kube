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

Setup the NGINX-ingress with a default backend...

```
$ kubectl apply -f manifests/ingress
```

Setup Let's Encrypt certificates using cert-manager (change the email in issuer-staging.yml and issuer-prod.yml)...

```
$ kubectl apply -f manifests/cert-manager/cert-manager.yaml # Wait till this is fully deployed
$ kubectl apply -f manifests/cert-manager/issuer-staging.yaml
$ kubectl apply -f manifests/cert-manager/issuer-prod.yaml
$ kubectl apply -f manifests/cert-manager/kuard-test.yaml # Use this to test if certificates are working
```

Setup Gitlab Integration...

```
$ kubectl cluster-info | grep 'Kubernetes master' | awk '/http/ {print $NF}'
$ kubectl get secrets
$ kubectl get secret <secret name> -o jsonpath="{['data']['ca\.crt']}" | base64 --decode
$ kubectl apply -f manifests/gitlab/
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep gitlab-admin | awk '{print $1}')
```

Setup rook-ceph storage...

```
$ kubectl apply -f manifests/storage/common.yaml
$ kubectl apply -f manifests/storage/operator.yaml # Make sure to wait till this is fully deployed
$ kubectl apply -f manifests/storage/cluster.yaml
$ kubectl apply -f manifests/storage/storageclass.yaml
$ kubectl apply -f manifests/storage/toolbox.yaml # Use this to test if storage is working
```

Setup jupyter-lab (For now you will need to access the logs to get the token to authenticate)...

```
$ kubectl apply -f manifests/jupyter # This also sets up the namespace 'thetrains'
```

Setup the metrics server (from within another directory)...

```
$ git clone https://github.com/kubernetes-sigs/metrics-server.git
$ kubectl apply -f manifests/metrics
```

Setup the dashboard...

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc6/aio/deploy/recommended.yaml
$ kubectl -n kubernetes-dashboard get secret
$ kubectl -n kubernetes-dashboard describe secrets <secret-token-name>
$ kubectl proxy # To forward the dashboard port
```

You can then go to the following and enter the token to authenticate...

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/