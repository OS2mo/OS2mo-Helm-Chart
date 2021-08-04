<!--
SPDX-FileCopyrightText: Magenta ApS

SPDX-License-Identifier: MPL-2.0
-->

# Getting started
**Goal**: Deploying and testing the MO stack on Kubernetes

Getting started is a multi-step process:

1. Installing required tools
2. Getting a Kubernetes Cluster
3. Testing Kubernetes Cluster access
4. Deploying OS2mo

## 1. Installing required tools
**Goal**: Acquiring the tools necessary to bring up the project.

We need 3 tools to bring up our Kubernetes Cluster:

* <a href="https://kind.sigs.k8s.io/">`kind`</a>
* <a href="https://kubernetes.io/docs/tasks/tools/">`kubectl`</a>
* <a href="https://helm.sh/">`helm`</a> 

### Getting `kind`
**Goal**: Ensuring that we have `kind` installed.

*Note: `kind` is only required for deploying a cluster on `localhost`.
       Thus it can be skipped if a cluster is already available.
       For more information, see step:
       [2: Getting a Kubernetes Cluster.](#2-getting-a-kubernetes-cluster)*

If you do not already have `kind` installed, now is the time to install it:
```shell
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
sudo chown root. /usr/local/bin/kind
```

### Getting kubectl
**Goal**: Ensuring that we have `kubectl` installed.

If you do not already have `kubectl` installed, now is the time to install it:
```shell
sudo apt-get update && sudo apt-get install -y kubectl
```

### Getting kubectl
**Goal**: Ensuring that we have `helm` installed.

If you do not already have `helm` installed, now is the time to install it:
```shell
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

## 2. Getting a Kubernetes Cluster
**Goal**: Being able to access a cluster using `kubectl`.

If a cluster is already available, jump to step:
[3. Testing Kubernetes Cluster access](#3-testing-kubernetes-cluster-access).

If a cluster must be built, contact whomever is responsible for builting it.

If a local cluster for development / testing is wanted, keep reading.

### Building a local cluster

The local cluster will be built using `kind`, but what is kind?

`kind` is a tool for running local Kubernetes clusters using Docker container "nodes".
Thus the only dependencies here, is that we already have a `docker`-daemon running.

#### Bringing up a cluster using kind
**Goal**: Installing a Kubernetes Cluster with Ingress on `localhost`.

Run `./start.sh` several times, until step:
[3. Testing Kubernetes Cluster access](#3-testing-kubernetes-cluster-access) is working.

The script does the following (if required):

* Creates a cluster using `kind create cluster`.
* Installs an `nginx` ingress controller.
* Deploys `kubernetes-secret-generator` as a cluster service.

At which point we have a functional Kubernetes cluster running on our docker-daemon.

## 3. Testing Kubernetes Cluster access
**Goal**: Ensuring that we have access to the Kubernetes Cluster.

At this point we should be able to list all pods in the cluster using:
```shell
$ kubectl get pods -A
```
On an empty cluster, the expected output is:
```
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-558bd4d5db-b587r                     1/1     Running   0          81s
kube-system          coredns-558bd4d5db-qb282                     1/1     Running   0          81s
kube-system          etcd-kind-control-plane                      1/1     Running   0          84s
kube-system          kindnet-gg4ng                                1/1     Running   0          81s
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          84s
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          84s
kube-system          kube-proxy-4g4fn                             1/1     Running   0          81s
kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          96s
local-path-storage   local-path-provisioner-547f784dff-ml2xm      1/1     Running   0          81s
```
Or similar, subject to the cluster in question.

## 4. Deploying OS2mo
**Goal**: Get OS2mo running on our Cluster

Simply run:
```shell
helm install chart/ --generate-name
```
After which OS2mo will be available on the cluster ingress.

If using the local `kind` setup, this will be:
<a href="http://localhost:80/">http://localhost:80/</a>.

If using another cluster use whatever method is available to access the ingress.

## 5. Getting data into OS2mo
**Goal**: Get data into OS2mo

At this point an empty OS2mo stack will be created.
Currently there are two ways of getting data into OS2mo:

1. Using the fixture loader
2. Using the data integrations

### 5.1 Using the fixture loader
**Goal**: Get data into OS2mo

Using the fixture loader is simple, simply set `load_fixture: true` as a values override:
```shell
helm install chart/ --generate-name --set load_fixture=true
```
Or using an `values-override.yaml` file
```shell
helm install chart/ --generate-name -f values-override.yaml
```
*Note: Use `helm upgrade` if a deployment has already been made.*

The fixture loader will run, and load the "Kolding Fixture" into the OS2mo instance.

### 5.2 Using the data integrations
**Goal**: Get data into OS2mo

The following section will be a generalized setup, as dataloading depends on the integration used.
However several steps are generic, and can be presented here.

#### Mapping Kubernetes settings to DIPEX's `settings.json`

Any configuration required in the `settings.json` file, should be added to the `dipex.settings`
mapping, either via `--set` or via the `values-override.yaml` file, as seen above.

Once this is done, update the deployment using:
```shell
helm secrets upgrade -f values-override.yaml
```
After this we should have a DIPEX deployment pod running.

Find its name, using `kubectl get pods`, for instance: `dipex-deployment-676d7798b6-bvmn9`.
Now attach to the dipex pod using:
```shell
kubectl exec -it dipex-deployment-ID_HERE -- /bin/bash
```
And generate the `settings.json` settings file using the following commands:
```shell
python3 tools/k8s_gen_settings.py /opt/settings/ /opt/secrets/ > settings/settings.json
```
At this point the various DIPEX integrations should be ready for use.

#### Creating root organisation

In order to actually use MO we need a root organisation.
This must manually be created from within the dipex container.

If not already inside the DIPEX container, drop inside it using:
```shell
kubectl exec -it dipex-deployment-ID_HERE -- /bin/bash
```
For details see the above section.

To setup a root organisation and some default classes use:
```shell
python3 tools/default_mo_setup.py
```

Now you *should* be ready to use dipex commands, such as:
```shell
python3 integrations/SD_Lon/sd_changed_at.py --init
```
or equivalently:
```shell
bash tools/job-runner.sh imports_sd_changed_at
```
