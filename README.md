# Kustomize - Introduction & Demo

This repository contains a collection of examples and use cases for Kustomize, a Kubernetes-native configuration management tool. 
These examples aim to demonstrate how Kustomize simplifies and enhances the configuration and deployment of applications in Kubernetes.

## What is Kustomize?

`kustomize` lets you customize raw, template-free YAML
files for multiple purposes, leaving the original YAML
untouched and usable as is.

`kustomize` targets kubernetes; it understands and can
patch [kubernetes style] API objects.  It's like
[`make`], in that what it does is declared in a file,
and it's like [`sed`], in that it emits edited text.

[Source](https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/README.md)

## kubectl integration

Since [kubectl v1.14][kubectl announcement] Kustomize is integrated in the cli. 

```sh
# View Resources found in a directory containing a kustomization file
kubectl kustomize <kustomization_directory>

# Apply those Resources
kubectl apply -k <kustomization_directory>
```

## Common folder structure

> ```
> ~/someApp
> ├── base
> │   ├── deployment.yaml
> │   ├── kustomization.yaml
> │   ├── service.yaml
> │   └── ...
> └── overlays
>     ├── development
>     │   ├── cpu_count.yaml
>     │   ├── kustomization.yaml
>     ├── production
>     │   ├── cpu_count.yaml
>     │   ├── kustomization.yaml
>     │   └── replica_count.yaml
>     └── ...
> ```

* `base` - Base configuration for Kustomize
* `overlays` - Specific environment configuration for Kustomize

An overlay is just another kustomization, referring to
the base, and referring to patches to apply to that
base.

This arrangement makes it easy to manage your
configuration with `git`.  The base could have files locally or
from an upstream repository managed by someone else.
The overlays could be in a repository you own.
Arranging the repo clones as siblings on disk avoids
the need for git submodules (though that works fine, if
you are a submodule fan).

## Demos

During the next section, we'll use the property `KUSTOMIZE_DEMO` pointing to the root of this project.

### Resources

This sample is here to showcase that it's super simple to onboard on Kustomize.
You just need to create a `kustomization.yaml` to declare the target resources, and you're done.

```sh
# Show the output
kubectl kustomize $KUSTOMIZE_DEMO/00_resources/base

# Apply
kubectl apply -k $KUSTOMIZE_DEMO/00_resources/base

# Check
kubectl run busybox --image=busybox --rm -it --restart=Never \
-- wget -qO- http://nginx.default.svc.cluster.local:8080

# Delete
kubectl delete -k $KUSTOMIZE_DEMO/00_resources/base
```

### Generator

In this sample we are going to use a configmap generator, a Kustomize feature.

```sh
# Show the output
kubectl kustomize $KUSTOMIZE_DEMO/01_generator/base

# Apply
kubectl apply -k $KUSTOMIZE_DEMO/01_generator/base

# Check
kubectl run busybox --image=busybox --rm -it --restart=Never \
-- wget -qO- http://nginx.default.svc.cluster.local:8080

# Delete
kubectl delete -k $KUSTOMIZE_DEMO/01_generator/base
```

> ℹ️ The hash appended at the end of the configmap will be updated every time the content of the configmap change. Hashing can be disabled.

> 💡 There is a secret generator too, for more info refer to [this](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/#generating-resources).


### Cross-cutting fields

In this sample we are going to define common stuff. 
We'll also start to use the overlay structure. We are now referencing the `01_generator/base/kustomization.yaml` we used previously as resource.

```sh
# Show the output
kubectl kustomize $KUSTOMIZE_DEMO/02_cross-cutting-fields/overlays/dev

# Apply
kubectl apply -k $KUSTOMIZE_DEMO/02_cross-cutting-fields/overlays/dev

# Check
kubectl run busybox --image=busybox --rm -it --restart=Never \
-- wget -qO- http://common-nginx-001.cross-cutting-fields.svc.cluster.local:8080

# Delete
kubectl delete -k $KUSTOMIZE_DEMO/02_cross-cutting-fields/overlays/dev
```

> 💡 For more info refer to [this](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/#setting-cross-cutting-fields).

### Overlays

In this sample, we'll look how to customize the deployment per environment.

```sh
# Show the output
kubectl kustomize $KUSTOMIZE_DEMO/03_overlay-features/overlays/dev

# Apply
kubectl apply -k $KUSTOMIZE_DEMO/03_overlay-features/overlays/dev

# Check
kubectl run busybox --image=busybox --rm -it --restart=Never \
-- wget -qO- http://nginx.dev.svc.cluster.local:8082

# Delete
kubectl delete -k $KUSTOMIZE_DEMO/03_overlay-features/overlays/dev
```

## Limitations

* Previous configmap is not deleted
* Kustomize do not automatically create the namespace defined in the Kustomization file. You need a namespace declaration in the resources.
* Be carreful with the namespace usage, it will be applied everywhere. This can be a problem for use cases where you want to deploy something in different namespaces (Prometheus ServiceMonitor is a good sample)

## Resources

https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/
https://github.com/kubernetes-sigs/kustomize/tree/master/examples




[`make`]: https://www.gnu.org/software/make
[`sed`]: https://www.gnu.org/software/sed
[DAM]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#declarative-application-management
[KEP]: https://github.com/kubernetes/enhancements/blob/master/keps/sig-cli/2377-Kustomize/README.md
[Kubernetes Code of Conduct]: code-of-conduct.md
[applied]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#apply
[base]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#base
[declarative configuration]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#declarative-application-management
[kubectl announcement]: https://kubernetes.io/blog/2019/03/25/kubernetes-1-14-release-announcement
[kubernetes documentation]: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
[kubernetes style]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#kubernetes-style-object
[kustomization]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#kustomization
[overlay]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#overlay
[overlays]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#overlay
[release page]: https://github.com/kubernetes-sigs/kustomize/releases
[resource]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#resource
[resources]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#resource
[sig-cli]: https://github.com/kubernetes/community/blob/master/sig-cli/README.md
[variants]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#variant