# Shipwright PoC with a customized Buildpacks `ClusterBuildStrategy`

[`Shipwright`](https://github.com/shipwright-io/build) is an extensible framework for building container images on Kubernetes.

## Why?

With Shipwright, developers get a simplified approach for building container images, by defining a minimal YAML that does not require
any previous knowledge of containers or container tooling. All you need is your source code in git and access to a container registry.

Shipwright supports any tool that can build container images in Kubernetes clusters, such as:

- [Kaniko](https://github.com/GoogleContainerTools/kaniko)
- [Cloud Native Buildpacks](https://buildpacks.io/)
- [BuildKit](https://github.com/moby/buildkit)
- [Buildah](https://buildah.io/)

## Try It!

* We assume you already have a Kubernetes cluster (v1.20+). If you don't, you can use [KinD](https://kind.sigs.k8s.io).

* We also require a Tekton installation (v0.27+). To install the newest supported version, run:

  ```bash
  kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.30.0/release.yaml
  ```

* Install the Shipwright deployment. To install the latest version, run:

  ```bash
  kubectl apply -f https://github.com/shipwright-io/build/releases/download/v0.8.0/release.yaml
  ```

* Install the Shipwright strategies. To install the latest version, run:

  ```bash
  kubectl apply -f ./samples/strategy.yaml
  ```

* Generate a secret to access your container registry, such as one on [Docker Hub](https://hub.docker.com/) or [Quay.io](https://quay.io/):

  ```bash
  REGISTRY_SERVER=https://index.docker.io/v1/ REGISTRY_USER=<your_registry_user> REGISTRY_PASSWORD=<your_registry_password>
  kubectl create secret docker-registry push-secret \
      --docker-server=$REGISTRY_SERVER \
      --docker-username=$REGISTRY_USER \
      --docker-password=$REGISTRY_PASSWORD  \
      --docker-email=<your_email>
  ```

* Create a *Build* object, replacing `<REGISTRY_ORG>` with the registry username your `push-secret` secret have access to:


  ```bash
  kubectl apply -f ./samples/build.yaml
  ```

To view the *Build* which you just created:

  ```
 $ kubectl get builds
 
  NAME                         REGISTERED   REASON      BUILDSTRATEGYKIND      BUILDSTRATEGYNAME              CREATIONTIME
  buildpack-typescript-build   True         Succeeded   ClusterBuildStrategy   buildpacks-typescript-v3       68s
  ```  

* Submit your *BuildRun*:

  ```bash
  kubectl apply -f ./samples/buildrun.yaml
  ```

* Wait until your *BuildRun* is completed and then you can view it as follows:

  ```
  $ kubectl get buildruns
  
  NAME                              SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
  buildpack-typescript-buildrun-xyzds   True        Succeeded   69s         2s
  ```

  or

  ```bash
  kubectl get buildrun --output name | xargs kubectl wait --for=condition=Succeeded --timeout=180s
  ```

* After your *BuildRun* is completed, check your container registry, you will find the new generated image uploaded there.
