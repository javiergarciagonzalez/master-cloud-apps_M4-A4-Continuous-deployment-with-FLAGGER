# Canary Release with Flagger
## Repo: master-cloud-apps_M4-A4-Continuous-deployment-with-FLAGGER

### 1. Generate Docker images for App's versions
First, we need to generate Docker images for all different versions of `LibraryApp`.


``` bash
> mvn spring-boot:build-image -Dspring-boot.build-image.imageName=javiergarciagon/mca-m4-a4-pr2:<version-number>
```

For every different version, a different version number has been used. These are: v1, v2, v3 & v4.

This will upload Docker images that contain LibraryApp's code and it will be created using repository ownner's Docker username (javiergarciagon) and its image name (mca-m4-a4-pr2). See Docker image in [DockerHub](https://hub.docker.com/repository/docker/javiergarciagon/mca-m4-a4-pr2).

Last step is to push all tags to the origin repository.
``` bash
> docker push --all-tags javiergarciagon/mca-m4-a4-pr2
```


### 2. Set up Minikube Cluster

```bash
# Start minikube cluster with desired addons and enough hardware resources
> minikube start --memory 16384 --cpus 4 --driver virtualbox --addons ingress --addons istio-provisioner --addons istio --addons metrics-server
```
### 3. Add Flagger to k8s cluster

``` bash
> helm repo add flagger https://flagger.app

> helm repo update

> kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artifacts/flagger/crd.yaml
> helm upgrade -i flagger flagger/flagger --namespace=istio-system --set crd.create=false --set meshProvider=istio --set metricsServer=http://prometheus:9090
```

### 4. Configure Flagger within the k8s cluster
``` bash
> kubectl create ns test

> kubectl label namespace test istio-injection=enabled

> kubectl config set-context --current --namespace=test
```

### 5. Add k8s files
``` bash
# First we add the DB because it is required by the app to start.
> kubectl apply -f ./flagger/mysql.yml

> kubectl apply -f ./flagger/libraryapp.yaml

> kubectl apply -f ./flagger/gateway.yaml
```

### 6. Start Canary Release
``` bash
# Apply canary yaml file to the k8s cluster
> kubectl apply -f ./flagger/canary.yml

# Watch Flagger's output while performing the canary release
> watch -n 1 'kubectl describe canary.flagger.app/libraryapp|tail'
```

### 7. Release follow up versions

Once Flagger has reached goal of 50% of traffic (weight) redirected to newest version, it is possible to release the next version.
``` bash
> kubectl set image deployment/library javiergarciagon/mca-m4-a4-pr2:<version-number>
```
Repeat this process for each one of the app's versions.