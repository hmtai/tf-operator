# Developer Guide

Tf-operator is currently at v1. The v1beta2 version will still be supported and is compatible with v1.

## Building the operator

Create a symbolic link inside your GOPATH to the location you checked out the code

```sh
mkdir -p ${go env GOPATH}/src/github.com/kubeflow
ln -sf ${GIT_TRAINING} ${go env GOPATH}/src/github.com/kubeflow/tf-operator
```

* GIT_TRAINING should be the location where you checked out https://github.com/kubeflow/tf-operator

Resolve dependencies (if you don't have dep install, check how to do it [here](https://github.com/golang/dep))

Install dependencies

```sh
dep ensure
```

Build it

```sh
go install github.com/kubeflow/tf-operator/cmd/tf-operator.v1
```

## Building all the artifacts.

[pipenv](https://docs.pipenv.org/) is recommended to manage local Python environment.
You can find setup information on their website.

To build the following artifacts:

* Docker image for the operator
* Helm chart for deploying it

You can run

```sh
# to setup pipenv you have to step into the directory where Pipfile is located
cd py/kubeflow/tf_operator
pipenv install
pipenv shell
cd ../..
python -m kubeflow.tf_operator.release local --registry=${REGISTRY}
```

* The docker image will be tagged into your registry
* The helm chart will be created in **./bin**

## Running the Operator Locally

Running the operator locally (as opposed to deploying it on a K8s cluster) is convenient for debugging/development.

### Run a Kubernetes cluster

First, you need to run a Kubernetes cluster locally. There are lots of choices:

- [local-up-cluster.sh in Kubernetes](https://github.com/kubernetes/kubernetes/blob/master/hack/local-up-cluster.sh)
- [minikube](https://github.com/kubernetes/minikube)

`local-up-cluster.sh` runs a single-node Kubernetes cluster locally, but Minikube runs a single-node Kubernetes cluster inside a VM. It is all compilable with the controller, but the Kubernetes version should be `1.8` or above.

Notice: If you use `local-up-cluster.sh`, please make sure that the kube-dns is up, see [kubernetes/kubernetes#47739](https://github.com/kubernetes/kubernetes/issues/47739) for more details.

### Configure KUBECONFIG and KUBEFLOW_NAMESPACE

We can configure the operator to run locally using the configuration available in your kubeconfig to communicate with
a K8s cluster. Set your environment:

```sh
export KUBECONFIG=$(echo ~/.kube/config)
export KUBEFLOW_NAMESPACE=$(your_namespace)
```

* KUBEFLOW_NAMESPACE is used when deployed on Kubernetes, we use this variable to create other resources (e.g. the resource lock) internal in the same namespace. It is optional, use `default` namespace if not set.

### Create the TFJob CRD

After the cluster is up, the TFJob CRD should be created on the cluster.

```bash
kubectl create -f ./examples/crd/crd-v1.yaml
```

### Run Operator

Now we are ready to run operator locally:

```sh
tf-operator
```

To verify local operator is working, create an example job and you should see jobs created by it.

```sh
cd ./examples/v1/dist-mnist
docker build -f Dockerfile -t kubeflow/tf-dist-mnist-test:1.0 .
kubectl create -f ./tf_job_mnist.yaml
```

## Go version

On ubuntu the default go package appears to be gccgo-go which has problems see [issue](https://github.com/golang/go/issues/15429) golang-go package is also really old so install from golang tarballs instead.

## Code Style

### Python

* Use [yapf](https://github.com/google/yapf) to format Python code
* `yapf` style is configured in `.style.yapf` file
* To autoformat code

  ```sh
  yapf -i py/**/*.py
  ```

* To sort imports

  ```sh
  isort path/to/module.py
  ```
