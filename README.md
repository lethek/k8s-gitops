<div align="center">
<img src="https://camo.githubusercontent.com/5b298bf6b0596795602bd771c5bddbb963e83e0f/68747470733a2f2f692e696d6775722e636f6d2f7031527a586a512e706e67" align="center" width="144px" height="144px"/>

### My home Kubernetes cluster ⛵

_... managed with Flux and Renovate_ 🤖

</div>
<br/>
<div align="center">

[![Discord](https://img.shields.io/discord/673534664354430999?style=for-the-badge&label=discord&logo=discord&logoColor=white)](https://discord.gg/k8s-at-home)
[![K3S](https://img.shields.io/badge/k3s-v1.23.3-brightgreen?style=for-the-badge&logo=kubernetes&logoColor=white)](https://k3s.io/)
[![Renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?style=for-the-badge&logo=renovatebot&logoColor=white)](https://github.com/renovatebot/renovate)
[![Lines of code](https://img.shields.io/tokei/lines/github/lethek/k8s-gitops?style=for-the-badge&color=brightgreen&label=lines&logo=codefactor&logoColor=white)](https://github.com/lethek/k8s-gitops/graphs/contributors)
  
</div>

---

## 📖 Overview
This is home to my personal Kubernetes cluster deployed with [Ansible](https://www.ansible.com) and [Terraform](https://www.terraform.io) backed by [Flux](https://toolkit.fluxcd.io/) and [SOPS](https://toolkit.fluxcd.io/guides/mozilla-sops/). [Renovate](https://github.com/renovatebot/renovate) also watches this Git repository and creates pull requests when it finds updates to Docker images, Helm charts, and other dependencies.

This repository is forked from the excellent [Kubernetes @Home template cluster](https://github.com/k8s-at-home/template-cluster-k3s).

- [Introduction](#-introduction)
- [Prerequisites](#-prerequisites)
- [Repository structure](#-repository-structure)
- [Lets go!](#-lets-go)
- [Post installation](#-post-installation)
- [Thanks](#-thanks)

## 👋 Introduction

The following components will be installed in the [k3s](https://k3s.io/) cluster by default.

Feel free to read up on any of these technologies before you get started to be more familiar with them.

- [cert-manager](https://cert-manager.io/) - SSL certificates - with Cloudflare DNS challenge
- [calico](https://www.tigera.io/project-calico/) - CNI (container network interface)
- [flux](https://toolkit.fluxcd.io/) - GitOps tool for deploying manifests from the `cluster` directory
- [hajimari](https://github.com/toboshii/hajimari) - start page with ingress discovery
- [kube-vip](https://kube-vip.io/) - layer 2 load balancer for the Kubernetes control plane
- [local-path-provisioner](https://github.com/rancher/local-path-provisioner) - default storage class provided by k3s
- [metallb](https://metallb.universe.tf/) - bare metal load balancer
- [reloader](https://github.com/stakater/Reloader) - restart pods when Kubernetes `configmap` or `secret` changes
- [reflector](https://github.com/emberstack/kubernetes-reflector) - mirror `configmap`s or `secret`s to other Kubernetes namespaces
- [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller) - upgrade k3s
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) - ingress controller
- [traefik](https://traefik.io) - another ingress controller (installed but not currently being used)

For provisioning the following tools will be used:

- [Ubuntu](https://ubuntu.com/download/server) - this is a pretty universal operating system that supports running all kinds of home related workloads in Kubernetes
- [Ansible](https://www.ansible.com) - this will be used to provision the Ubuntu operating system to be ready for Kubernetes and also to install k3s
- [Terraform](https://www.terraform.io) - in order to help with the DNS settings this will be used to provision an already existing Cloudflare domain and DNS settings

## 📝 Prerequisites

### 💻 Systems

- One or more nodes with a fresh install of [Ubuntu Server 20.04](https://ubuntu.com/download/server). These nodes can be bare metal or VMs.
- A [Cloudflare](https://www.cloudflare.com/) account with a domain, this will be managed by Terraform.
- Some experience in debugging problems and a positive attitude ;)

### 🔧 Tools

📍 You should install the below CLI tools on your workstation. Make sure you pull in the latest versions.

#### Required

| Tool                                               | Purpose                                                                                                                                 |
|----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| [ansible](https://www.ansible.com)                 | Preparing Ubuntu for Kubernetes and installing k3s                                                                                      |
| [direnv](https://github.com/direnv/direnv)         | Exports env vars based on present working directory                                                                                     |
| [flux](https://toolkit.fluxcd.io/)                 | Operator that manages your k8s cluster based on your Git repository                                                                     |
| [age](https://github.com/FiloSottile/age)          | A simple, modern and secure encryption tool (and Go library) with small explicit keys, no config options, and UNIX-style composability. |
| [go-task](https://github.com/go-task/task)         | A task runner / simpler Make alternative written in Go                                                                                  |
| [ipcalc](http://jodies.de/ipcalc)                  | Used to verify settings in the configure script                                                                                         |
| [jq](https://stedolan.github.io/jq/)               | Used to verify settings in the configure script                                                                                         |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | Allows you to run commands against Kubernetes clusters                                                                                  |
| [sops](https://github.com/mozilla/sops)            | Encrypts k8s secrets with Age                                                                                                           |
| [terraform](https://www.terraform.io)              | Prepare a Cloudflare domain to be used with the cluster                                                                                 |

#### Optional

| Tool                                                   | Purpose                                                  |
|--------------------------------------------------------|----------------------------------------------------------|
| [helm](https://helm.sh/)                               | Manage Kubernetes applications                           |
| [kustomize](https://kustomize.io/)                     | Template-free way to customize application configuration |
| [pre-commit](https://github.com/pre-commit/pre-commit) | Runs checks pre `git commit`                             |
| [gitleaks](https://github.com/zricethezav/gitleaks)    | Scan git repos (or files) for secrets                    |
| [prettier](https://github.com/prettier/prettier)       | Prettier is an opinionated code formatter.               |

### ⚠️ pre-commit

It is advisable to install [pre-commit](https://pre-commit.com/) and the pre-commit hooks that come with this repository.
[sops-pre-commit](https://github.com/k8s-at-home/sops-pre-commit) and [gitleaks](https://github.com/zricethezav/gitleaks) will check to make sure you are not by accident committing your secrets un-encrypted.

After pre-commit is installed on your machine run:

```sh
task pre-commit:init
```
**Remember to run this on each new clone of the repository for it to have effect.**

Commands are of interest, for learning purposes:

This command makes it so pre-commit runs on `git commit`, and also installs environments per the config file.
```
pre-commit install --install-hooks
```
This command checks for new versions of hooks, though it will occasionally make mistakes, so verify its results.
```
pre-commit autoupdate
```

## 📂 Repository structure

The Git repository contains the following directories under `cluster` and are ordered below by how Flux will apply them.

- **base** directory is the entrypoint to Flux
- **crds** directory contains custom resource definitions (CRDs) that need to exist globally in your cluster before anything else exists
- **core** directory (depends on **crds**) are important infrastructure applications (grouped by namespace) that should never be pruned by Flux
- **apps** directory (depends on **core**) is where your common applications (grouped by namespace) could be placed, Flux will prune resources here if they are not tracked by Git anymore

```
cluster
├── apps
│   ├── default
│   ├── networking
│   └── system-upgrade
├── base
│   └── flux-system
├── core
│   ├── cert-manager
│   ├── metallb-system
│   ├── namespaces
│   └── system-upgrade
└── crds
    └── cert-manager
```

## 🚀 Lets go!

Very first step will be to create a new repository by clicking the **Use this template** button on this page.

Clone the repo to you local workstation and `cd` into it.

📍 **All of the below commands** are run on your **local** workstation, **not** on any of your cluster nodes.

### 🔐 Setting up Age

📍 Here we will create a Age Private and Public key. Using SOPS with Age allows us to encrypt and decrypt secrets.

1. Create a Age Private / Public Key

```sh
age-keygen -o age.agekey
```

2. Set up the directory for the Age key and move the Age file to it

```sh
mkdir -p ~/.config/sops/age
mv age.agekey ~/.config/sops/age/keys.txt
```

3. Export the `SOPS_AGE_KEY_FILE` variable in your `bashrc`, `zshrc` or `config.fish` and source it, e.g.

```sh
export SOPS_AGE_KEY_FILE=~/.config/sops/age/keys.txt
source ~/.bashrc
```

4. Fill out the Age public key in the `.config.env` under `BOOTSTRAP_AGE_PUBLIC_KEY`, **note** the public key should start with `age`...

### ☁️ Global Cloudflare API Key

In order to use Terraform and `cert-manager` with the Cloudflare DNS challenge you will need to create a API key.

1. Head over to Cloudflare and create a API key by going [here](https://dash.cloudflare.com/profile/api-tokens).

2. Under the `API Keys` section, create a global API Key.

3. Use the API Key in the configuration section below.

📍 You may wish to update this later on to a Cloudflare **API Token** which can be scoped to certain resources. I do not recommend using a Cloudflare **API Key**, but for the purposes of this template it is easier getting started without having to define which scopes and resources are needed. For more information see the [Cloudflare docs on API Keys and Tokens](https://developers.cloudflare.com/api/).

### 📄 Configuration

📍 The `.config.env` file contains necessary configuration that is needed by Ansible, Terraform and Flux.

📍 It is suggested to use **three control plane nodes**. If you **only need a single control plane node**, make sure **you update** `./provision/ansible/inventory/group_vars/kubernetes/k3s.yml` and set `k3s_use_unsupported_config` to `true`

1. Copy the `.config.sample.env` to `.config.env` and start filling out all the environment variables. **All are required** and read the comments they will explain further what is required.

2. Once that is done, verify the configuration is correct by running `./configure.sh --verify`

3. If you do not encounter any errors run `./configure.sh` to start having the script wire up the templated files and place them where they need to be.

### ⚡ Preparing Ubuntu with Ansible

📍 Here we will be running a Ansible Playbook to prepare Ubuntu for running a Kubernetes cluster.

📍 Nodes are not security hardened by default, you can do this with [dev-sec/ansible-collection-hardening](https://github.com/dev-sec/ansible-collection-hardening) or something similar.

1. Ensure you are able to SSH into you nodes from your workstation with using your private ssh key. This is how Ansible is able to connect to your remote nodes.

2. Install the deps by running `task ansible:deps`

3. Verify Ansible can view your config by running `task ansible:list`

4. Verify Ansible can ping your nodes by running `task ansible:adhoc:ping`

5. Finally, run the Ubuntu Prepare playbook by running `task ansible:playbook:ubuntu-prepare`

6. If everything goes as planned you should see Ansible running the Ubuntu Prepare Playbook against your nodes.

### ⛵ Installing k3s with Ansible

📍 Here we will be running a Ansible Playbook to install [k3s](https://k3s.io/) with [this](https://galaxy.ansible.com/xanmanning/k3s) wonderful k3s Ansible galaxy role. After completion, Ansible will drop a `kubeconfig` in `./provision/kubeconfig` for use with interacting with your cluster with `kubectl`.

📍 Once more over, it is suggested to use **three control plane nodes**. If you **only need a single control plane node**, make sure **you update** `./provision/ansible/inventory/group_vars/kubernetes/k3s.yml` and set `k3s_use_unsupported_config` to `true`

1. Verify Ansible can view your config by running `task ansible:list`

2. Verify Ansible can ping your nodes by running `task ansible:adhoc:ping`

3. Run the k3s install playbook by running `task ansible:playbook:k3s-install`

4. If everything goes as planned you should see Ansible running the k3s install Playbook against your nodes.

5. Verify the nodes are online

```sh
kubectl --kubeconfig=./provision/kubeconfig get nodes
# NAME           STATUS   ROLES                       AGE     VERSION
# k8s-0          Ready    control-plane,master      4d20h   v1.21.5+k3s1
# k8s-1          Ready    worker                    4d20h   v1.21.5+k3s1
```

### ☁️ Configure Cloudflare DNS with Terraform

📍 Review the Terraform scripts under `./provision/terraform/cloudflare/` and make sure you understand what it's doing (no really review it). If your domain already has existing DNS records be sure to export those DNS settings before you continue. Ideally you can update the terraform script to manage DNS for all records if you so choose to.

1. Pull in the Terraform deps by running `task terraform:init:cloudflare`

2. Review the changes Terraform will make to your Cloudflare domain by running `task terraform:plan:cloudflare`

3. Finally have Terraform execute the task by running `task terraform:apply:cloudflare`

If Terraform was ran successfully and you have port forwarded `80` and `443` in your router to the `${BOOTSTRAP_LB_NGINX_ADDR}` IP, head over to your browser and you _should_ be able to access `https://hajimari.${BOOTSTRAP_CLOUDFLARE_DOMAIN}`!

### 🔹 GitOps with Flux

📍 Here we will be installing [flux](https://toolkit.fluxcd.io/) after some quick bootstrap steps.

1. Verify Flux can be installed

```sh
flux --kubeconfig=./provision/kubeconfig check --pre
# ► checking prerequisites
# ✔ kubectl 1.21.5 >=1.18.0-0
# ✔ Kubernetes 1.21.5+k3s1 >=1.16.0-0
# ✔ prerequisites checks passed
```

2. Pre-create the `flux-system` namespace

```sh
kubectl --kubeconfig=./provision/kubeconfig create namespace flux-system --dry-run=client -o yaml | kubectl --kubeconfig=./provision/kubeconfig apply -f -
```

3. Add the Age key in-order for Flux to decrypt SOPS secrets

```sh
cat ~/.config/sops/age/keys.txt |
    kubectl --kubeconfig=./provision/kubeconfig \
    -n flux-system create secret generic sops-age \
    --from-file=age.agekey=/dev/stdin
```

📍 Variables defined in `./cluster/base/cluster-secrets.sops.yaml` and `./cluster/base/cluster-settings.yaml` will be usable anywhere in your YAML manifests under `./cluster`

4. **Verify** the `./cluster/base/cluster-secrets.sops.yaml` and `./cluster/core/cert-manager/secret.sops.yaml` files are **encrypted** with SOPS

5. If you verified all the secrets are encrypted, you can delete the `tmpl` directory now

6.  Push you changes to git

```sh
git add -A
git commit -m "initial commit"
git push
```

7. Install Flux

📍 Due to race conditions with the Flux CRDs you will have to run the below command twice. There should be no errors on this second run.

```sh
kubectl --kubeconfig=./provision/kubeconfig apply --kustomize=./cluster/base/flux-system
# namespace/flux-system configured
# customresourcedefinition.apiextensions.k8s.io/alerts.notification.toolkit.fluxcd.io created
# ...
# unable to recognize "./cluster/base/flux-system": no matches for kind "Kustomization" in version "kustomize.toolkit.fluxcd.io/v1beta1"
# unable to recognize "./cluster/base/flux-system": no matches for kind "GitRepository" in version "source.toolkit.fluxcd.io/v1beta1"
# unable to recognize "./cluster/base/flux-system": no matches for kind "HelmRepository" in version "source.toolkit.fluxcd.io/v1beta1"
# unable to recognize "./cluster/base/flux-system": no matches for kind "HelmRepository" in version "source.toolkit.fluxcd.io/v1beta1"
# unable to recognize "./cluster/base/flux-system": no matches for kind "HelmRepository" in version "source.toolkit.fluxcd.io/v1beta1"
# unable to recognize "./cluster/base/flux-system": no matches for kind "HelmRepository" in version "source.toolkit.fluxcd.io/v1beta1"
```

8. Verify Flux components are running in the cluster

```sh
kubectl --kubeconfig=./provision/kubeconfig get pods -n flux-system
# NAME                                       READY   STATUS    RESTARTS   AGE
# helm-controller-5bbd94c75-89sb4            1/1     Running   0          1h
# kustomize-controller-7b67b6b77d-nqc67      1/1     Running   0          1h
# notification-controller-7c46575844-k4bvr   1/1     Running   0          1h
# source-controller-7d6875bcb4-zqw9f         1/1     Running   0          1h
```

🎉 **Congratulations** you have a Kubernetes cluster managed by Flux, your Git repository is driving the state of your cluster.

## 📣 Post installation

### 👉 Troubleshooting

Our [wiki](https://github.com/k8s-at-home/template-cluster-k3s/wiki) is a good place to start troubleshooting issues. If that doesn't cover your issue, start a new thread in the #support channel on our [Discord](https://discord.gg/k8s-at-home).

### 🤖 Integrations

Our Check out our [wiki](https://github.com/k8s-at-home/template-cluster-k3s/wiki) for more integrations!

## ❔ What's next

The world is your cluster, try installing another application or if you have a NAS and want storage back by that check out the helm charts for [democratic-csi](https://github.com/democratic-csi/democratic-csi), [csi-driver-nfs](https://github.com/kubernetes-csi/csi-driver-nfs) or [nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner).

If you plan on exposing your ingress to the world from your home. Checkout [our rough guide](https://docs.k8s-at-home.com/guides/dyndns/) to run a k8s `CronJob` to update DDNS.

## 🤝 Thanks

Thanks to all the people who donate their time to the [Kubernetes @Home](https://github.com/k8s-at-home/) community. Big shout out to all the authors and contributors to the projects used in this repository.
