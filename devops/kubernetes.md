# Kubernetes

## Kind

Kind is a tool for running local Kubernetes clusters (in opposition to using a cloud provider).

Project URL: https://kind.sigs.k8s.io/

Create cluster:
```sh
kind create cluster -n development
kind create cluster -n production
```

Use `-n` flag to inform the name for the cluster.

### Debug cluster creation

When cluster creation fails, you can rerun the `kind create` command with the `--retain` in order to retain the cluster for debuging.

`kind export logs` will export logs files to a temporary folder indicated on the output of the command.

### Fix: too many files open

I had some problems creating a third cluster on my machine with Kind. Found out that I had ran out of `inotify` resources.

Found the error message `Failed to create control group inotify object: Too many open files` on the logs exported form kind in the `serial.log` file.

Increasing the variables `fs.inotify.max_user_watches` and `fs.inotify.max_user_instances` did the trick for me. More on this on the link below.

Source: https://kind.sigs.k8s.io/docs/user/known-issues/#pod-errors-due-to-too-many-open-files

## kubectl

The `kubectl` command line tool is the main tool for interacting with the clusters.

### Context

Each cluster is assigned to a context. The context contains information necessary for comunicating with the cluster, and, in a pratical way, is used to identify the cluster.

Display all contexts and indicates which one is active.
```sh
kubectl config get-contexts
```

Switch context:
```sh
kubectl config use-context kind-production
```

One can also use the `--context` flag to indicate the context of a operation (default context is ignored).
```sh
kubectl get all --context kind-development
```

## Kustomize

Kustomize is a standalone tool that got integrated into `kubectl` command.

Project URL: https://github.com/kubernetes-sigs/kustomize

Docs on: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/

You can customize Kubernetes objects with Kustomize. You need to write a kustomization file (`kustomization.yaml`) that indicates the YAML resources (services, deployments, etc) and the customizations to be done to those resorces (e.g. add a common label). You will then generate resource configuration (YAML files) with the constumizations applied (original sources are kept untouched).

Example: given a service.yaml and a deployment.yaml resource files, the following kustomization.yaml will add new resources with the `app: myapp` labels.

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
commonLabels:
  app: myapp
resources:
  - deployment.yaml
  - service.yaml
```

Generate resource config with `kubectl kustomize .` on the folder containing those files. Or apply them directly with `kubectl apply -k .` on the same folder.


### Overlays

Overlays are useful to generate variants with a common base. Often used to generate per-environment config resources.

Example: generate resource files for development and production environments with differente number of replicas.

File structure:
```
~/my-app
├── base
|     ├── kustomization.yaml
|     ├── service.yaml
|     └── deployment.yaml
├── development
|     ├── kustomization.yaml
|     ├── namespace.yaml
|     └── deployment-patch.yaml
└── production
      ├── kustomization.yaml
      ├── namespace.yaml
      └── deployment-patch.yaml
```

All common configuration are defined in the base folder. Specifics are defined on kustomization files on folders for each environment.

~/my-app/development/kustomization.yaml
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../base
- namespace.yaml

namespace: development
namePrefix: dev-

patches:
- path: deployment-patch.yaml
```

~/my-app/development/deployment-patch.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: podinfo
spec:
  replicas: 2
```

The kustomization file must indicate on the `resources` field the folder with the base `kustomization.yaml` and other configuration resources with specifics (e.g. the namespace.yaml defining the environment specific namespace).

There are various fields for customizing the resources, like the `namespace` for changing the namespace of the objects and `namePrefix` for prefixing the object names, shown on the example above. See more on https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/#kustomize-feature-list.

Patches indicated on the `patches` field are also applied, overwriting the fields values with those defined on the patches. Every file indicated as a patch must resolve to a Kubernetes object. In the above example the number of replicas of the Deployment object is set to 2 for the development environment.

Running the Kustomize on the `development` folder will generate sources with the base configurations plus the configurations on this folder.


## Flux

Flux is a tool for keeping clusters in synch with it's source configuration.

Project URL: https://fluxcd.io/

That means that when the source configuration is changed, Flux will update the cluster state to match it. In other words, Flux will watch for changes in the .yaml sources and apply them on the cluster.

Example: the spec.replicas field is modified from 2 to 3 on the deployment.yaml file that describes a Deployment object. When the change is pushed to the Git repository, Flux will identify the change and apply the intended increase of replicas on the cluster.

### Flux useful commands

Bootstrap:
```sh
flux bootstrap github --owner=RenanMarques --repository=my-repository --path=clusters/development
```
This command will set up Flux in the repository.
There are other venders like bitbucket, gitlab, git, etc.

Display flux logs:
```sh
flux logs
```

Display events from Flux. Useful for tracking reconciliations.
```sh
flux events flux-system/kustomization
```
