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
