# K8s to Local Minikube
1. [Remember](#Remember)
2. [Install](#Install)
    1. [Docker-cli and Docker-compose](#Docker-cli-and-Docker-compose)
    2. [Minikube](#Minikube)
    3. [Kubectl](#Kubectl)
    4. [VirtualBox](#VirtualBox)
3. [Create a minikube machine](#Create-a-minikube-machine)
    1. [Uninstall Hiper-V](#Uninstall-Hiper-V)
    2. [Start minikube](#Create-a-minikube)
    3. [Possible errors](#Possible-errors)
4. [Configure](#Configure)
    1. [Enable addons](#Enable-addons)
    2. [Tag minikube node](#Tag-minikube-node-(only-if-you-have-pods-attached-to-a-group-of-nodes))
    3. [Create k8s registry](#Create-k8s-registry)
    4. [Get access to the registry ](#get-access-to-the-registry-outside-localhost5000)
    5. [Check connectivity to the registry](#Check-connectivity-to-the-registry)
5. [Prepare your terminal to use minikube](#Prepare-your-terminal-to-use-minikube)


## Remember 
Test deployment of helm charts to your local environment.
*Remember*

- change DNS durint tests & after stop minikube
- minikube is no compatible with docker-compose
- minikube don't have fixed ip (usually on 192.168.99.100)
- minikube is no friend of docker-toolbox (I needed to reboot my machine few times)

## **Install**
We need to install all of this:
* Docker-cli , docker-compose /docker for windows (I prefer docker-cli and docker-compose only)
* Minikube
* Kubectl
* VirtualBox

### Docker-cli and Docker-compose
Install **Choco (https://chocolatey.org/install)** and with that install **docker-cli** and **docker-compose** 
Open a administrator powershell window and run:
```
	$ Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
	$ choco install docker-cli
	$ choco install docker-compose
```
### Minikube
You can install Minikube in different ways (https://kubernetes.io/es/docs/tasks/tools/install-minikube/#instalar-minikube) but I prefer to download the .exe and add it to de path by myselft.
```
Download https://storage.googleapis.com/minikube/releases/v1.2.0/minikube-windows-amd64.exe
Rename minikube-windows-amd64.exe to  minikube.exe
Create a folder in C:\minikube and move it the minikube.exe inside
Add the path to the windows environment variables:
[image]
```
### Kubectl
You can install Kubectl in different ways (https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows) but I prefer to download the .exe and add it to de path by myselft
 ```
Download https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe
Move it the kubectl.exe inside the C:\minikube folder
Now, you don't need to add the path to the variables because you did it before.
```
### VirtualBox
Download  https://www.virtualbox.org/wiki/Downloads and install it.

## **Create a minikube machine**
### Uninstall Hiper-V

Before create a minikube machine make sure that you have the Hiper-v uninstalled.
If you have a Hiper-v error go to windows search -> Activar o desactivar las características de windows-> Hiper-v-> unselect -> Aceptar.
[image]
Why that? Hiper-v and Virtual Box are hypervisors and windows only one of them for running your minikube machine. In this example we are using Virtual Box. So you need to remove Hiper-V. If you want to use Hiper-V there are a lot of posts for do it.(https://medium.com/@mudrii/kubernetes-local-development-with-minikube-on-hyper-v-windows-10-75f52ad1ed42) 
### Create a minikube
Sometimes this command doesnt work well. I needed to delete my minikube machine and reboot my pc a few times :D. So be patient my friend.
```
$ minikube start --memory 2048 
```
### Possible errors
If you have a Error: Call to WHvSetupPartition  remove the minikube machine and do it again.
```
$ minikube stop && minikube delete
```
If you have a Error:  “VT-x is not available. (VERR_VMX_NO_VMX)” -> remove hiper-v and reboot the pc. (twice reboots if its necesary, i needed to do it)


## Configure
### Enable addons 
We need to enable some addons for have permission to see the k8 cluster outside the minikube machine.
```
$ minikube addons enable ingress
$ minikube addons enable dashboard
```
### Tag minikube node (only if you have pods attached to a group of nodes)
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#create-a-pod-that-gets-scheduled-to-specific-node
``` 
$ kubectl label nodes minikube kops.k8s.io/instancegroup=nodes3 
```
### Create k8s registry
We need somewhere for upload our docker images, so we are going to create a docker private registry inside Kubernetes.
```
$ kubectl create -f https://gist.githubusercontent.com/coco98/b750b3debc6d517308596c248daf3bb1/raw/6efc11eb8c2dce167ba0a5e557833cc4ff38fa7c/kube-registry.yaml

```
### Get access to the registry outside (localhost:5000)
**In git bash**
```
$ kubectl port-forward --namespace kube-system $(kubectl get pods -n kube-system | grep kube-registry-v0 | awk '{print $1;}') 5000:5000 > /dev/null 2>&1 &

```
### Check connectivity to the registry

```
$ curl -X GET localhost:5000/v2/_catalog
```

## Prepare your terminal to use minikube
Everytime we open a terminal we need to attach our docker-cli to the minikube docker-env because if not, we can't upload docker images to our k8 registry.
Powershell:
```
$ minikube docker-env | Invoke-Expression
```
Git bash
```
$ eval $(minikube docker-env)
```
