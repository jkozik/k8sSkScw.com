# k8sSkScw.com
sancapweather.com, skaffold orchestrated Dockerbuild, image push, k8s deployment

# Fork of two repositories
This repository is a combination of [InstallSCW.com](https://github.com/jkozik/InstallSCW.com) and [k8sScw.com](https://github.com/jkozik/k8sScw.com).  These repositories are valueable because they show how the docker image was assembled and how it was deployed in a kubernetes cluster.  

This repository takes the union of these two repositories and wraps the [skaffold](https://skaffold.dev/) to build, push and deploy the sancapweather.com application. 

# skaffold.yaml
Here's the skaffold.yaml file that builds the Dockerfile, pushes the result to the docker.io repository, then following the manifests in the k8s directory to deploy the image into the kubernetes image.
```
apiVersion: skaffold/v2beta20
kind: Config
metadata:
  name: installscw.com
build:
  artifacts:
  - image: jkozik/scw.com
    docker:
#      noCache=true to rebuild everything.
#      noCache: true
```

Note: most of the time this works.  But if you want to repull everything, the way Dockerfiles work, you need to pass it a no-cache option.  Uncomment the noCache: true line if this case arises.  For me, this case comes up if the weather software script provider, http://saratoga-weather.com, issues a new release. 

# Defaults
## skaffold
On my host that runs kubectl, I installed skaffold from the root account following the [skaffold installation directions](https://skaffold.dev/docs/install/).

SKaffold needs to be told which image repository to upload to. I use my docker one.
```
skaffold config set default-repo docker.io/jkozik
```
Verify 
```
[jkozik@dell2 k8sSkScw.com]$ skaffold config list
skaffold config:
kube-context: kubernetes-admin@kubernetes
default-repo: docker.io/jkozik
```

## kubectl
The skaffold tool expects that kubectl is installed and pointing to a kubernetes cluster. I am not using minikube, I have my own kubeadm-config'd cluster with 3 nodes on my home LAN. 
```
[jkozik@dell2 k8sSkScw.com]$ kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
[jkozik@dell2 k8sSkScw.com]$
```

# skaffold run -- build, push and deploy
Here's the basic flow:
```
[jkozik@dell2 ~]$ git clone https://github.com/jkozik/k8sSkScw.com
Cloning into 'k8sSkScw.com'...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 18 (delta 0), reused 12 (delta 0), pack-reused 0
Receiving objects: 100% (18/18), 3.66 MiB | 17.70 MiB/s, done.

[jkozik@dell2 k8sSkScw.com]$ kubectl get pod scwcom-67f59694f7-bl2qz
NAME                      READY   STATUS    RESTARTS   AGE
scwcom-67f59694f7-bl2qz   1/1     Running   0          4m3s
```
As a reminder, one can get into the pod using the following command:
```
[jkozik@dell2 k8sSkScw.com]$ kubectl exec -it scwcom-67f59694f7-bl2qz  -- /bin/bash
root@scwcom-67f59694f7-bl2qz:/var/www/html# pwd
/var/www/html
```
Also, I used the following commands for troubleshooting purposes.
```
skaffold run -v TRACE
skaffold run -v DEBUG
skaffold build
skaffold deploy
```
