
## Pre-requisites

You must have a the ability to run root containers in a cluster
with at least 12 vCPUs and 45G of memory and a minumum of three nodes.
You will also need to provide a docker compatable registry. 

## Configure your Registry
In order to make this install simple to walk through we recomend you 
store your docker registry details in a an environment variable. 

```
export KF_REGISTRY=gcr.io/<PROJECT_ID>
```


## Install Istio && Knative

Install istio CRDs and deploy pods and label the default namespace. 
```
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.5.0/istio-crds.yaml && \
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.5.0/istio.yaml && \
kubectl label namespace default istio-injection=enabled
```

Install knative CRDs.
```
kubectl apply --selector knative.dev/crd-install=true \
--filename https://github.com/knative/serving/releases/download/v0.5.0/serving.yaml \
--filename https://github.com/knative/build/releases/download/v0.5.0/build.yaml \
--filename https://github.com/knative/eventing/releases/download/v0.5.0/release.yaml \
--filename https://github.com/knative/eventing-sources/releases/download/v0.5.0/eventing-sources.yaml \
--filename https://github.com/knative/serving/releases/download/v0.5.0/monitoring.yaml \
```

Install Knative PODs
```
--filename https://raw.githubusercontent.com/knative/serving/v0.5.0/third_party/config/build/clusterrole.yaml && \
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.5.0/serving.yaml \
--filename https://github.com/knative/build/releases/download/v0.5.0/build.yaml \
--filename https://github.com/knative/eventing/releases/download/v0.5.0/release.yaml \
--filename https://github.com/knative/eventing-sources/releases/download/v0.5.0/eventing-sources.yaml \
--filename https://github.com/knative/serving/releases/download/v0.5.0/monitoring.yaml \
--filename https://raw.githubusercontent.com/knative/serving/v0.5.0/third_party/config/build/clusterrole.yaml
```


## Upload buildpacks
Buildpacks are provided by the operator and can be uploaded to Knative using the
CLI. A set of buidpacks is included in this repo. Change into the `buildpack-samples`
directory run the following command. 

```
kf upload-buildpacks --container-registry $KF_REGISTRY
```

## Push your first app
At this point you are ready to deploy your first app using `kf`. Run the following command 
from the `sample-apps\helloworld` directory. 

```
kf push helloworld --container-registry $KF_REGISTRY
```

## Install the service catalog
To install the service catalog you will need to clone the svcat repo, and install
helm into your cluster. 

```
git clone https://github.com/kubernetes-incubator/service-catalog
cd service-catalog
kubectl create serviceaccount --namespace kube-system tiller 
kubectl create clusterrolebinding tiller-cluster-rule \
--clusterrole=cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```

Once tiller is running you can install svcat with the following command
from the `service-catalog/charts/catalog` directory.

```
helm install . --name catalog --namespace catalog
```

You should be able to see an empty marketplace at this point by running.

```
kf marketplace
```

## Installig a broker
Once you have the service catalog you'll want to install a service
broker. To do this you'll need the svcat CLI tool, `sc`. This can be
install via homebrew. 

```
brew update
brew install kubernetes-service-catalog-client
sc add-gcp-broker
```