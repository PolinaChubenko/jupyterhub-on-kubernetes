# jupyterhub-on-kubernetes

## Helm
Installing helm https://helm.sh/docs/intro/install/

```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

## kubectl
Install kubectl binary with curl on Linux https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

Download the latest release with the command:
```curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"```

Validate the binary (optional)
Download the kubectl checksum file:
```curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"```

Validate the kubectl binary against the checksum file:
```echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check```

If valid, the output is:
```kubectl: OK```

Install kubectl
```sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl```

Test to ensure the version you installed is up-to-date:
```kubectl version --client```

## Minikube or kind
Установка Minikube для локального подъема kubernetes https://kubernetes.io/ru/docs/tasks/tools/install-minikube/

Установка kind для локального подъема kubernetes: https://kind.sigs.k8s.io/docs/user/quick-start/

Creating a Kubernetes cluster:
```kind create cluster```
or 
```kind create cluster --name kind-2```

To list your kind clusters: `kind get clusters`

In order to interact with a specific cluster, you only need to specify the cluster name as a context in kubectl:
```
kubectl cluster-info --context kind-kind
kubectl cluster-info --context kind-kind-2
```

## Кластеры и kubectl

Если с помощью kubectl вы работаете с несколькими кластерами, убедитесь, что вы выбрали правильный контекст:
```
kubectl config get-contexts
CURRENT   NAME          CLUSTER       AUTHINFO      NAMESPACE
          kind-kind     kind-kind     kind-kind     
*         kind-kind-2   kind-kind-2   kind-kind-2
```
Звездочка означает, что мы подключены к кластеру kind-kind-2
Чтобы переключиться на другой кластер, введите: `kubectl config use-context kind-kind`  

## Jupyterhub in Kubernetes
Running Jupyterhub in Kubernetes https://citizix.com/how-to-run-jupyterhub-in-kubernetes/

Created dir and empty config.yaml there

Make Helm aware of the JupyterHub Helm chart repository so you can install the JupyterHub chart from it without having to use a long URL name.

```
helm repo add jupyterhub https://hub.jupyter.org/helm-chart/
helm repo update
```
This should show output like:
```
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
...Successfully got an update from the "jupyterhub" chart repository
Update Complete. ⎈ Happy Helming!⎈
```
To see the available versions of JupyterHub run
```
helm search repo jupyterhub
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION                                       
jupyterhub/jupyterhub	3.2.1        	4.0.2      	Multi-user Jupyter installation                   
jupyterhub/pebble    	1.0.1        	v2.3.1     	This Helm chart bootstraps Pebble: an ACME serv...
```
Now install the chart configured by your config.yaml by running this command from the directory that contains your config.yaml:
```
helm upgrade --cleanup-on-fail \
  --install jupyterhub jupyterhub/jupyterhub \
  --namespace jupyterhub \
  --create-namespace \
  --version=3.2.1 \
  --values config.yaml
```
Выхлоп будет следующий
```
Release "jupyterhub" does not exist. Installing it now.
NAME: jupyterhub
LAST DEPLOYED: Sun Feb 18 13:06:08 2024
NAMESPACE: jupyterhub
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
.      __                          __                  __  __          __
      / / __  __  ____    __  __  / /_  ___    _____  / / / / __  __  / /_
 __  / / / / / / / __ \  / / / / / __/ / _ \  / ___/ / /_/ / / / / / / __ \
/ /_/ / / /_/ / / /_/ / / /_/ / / /_  /  __/ / /    / __  / / /_/ / / /_/ /
\____/  \__,_/ / .___/  \__, /  \__/  \___/ /_/    /_/ /_/  \__,_/ /_.___/
              /_/      /____/

       You have successfully installed the official JupyterHub Helm chart!

### Installation info

  - Kubernetes namespace: jupyterhub
  - Helm release name:    jupyterhub
  - Helm chart version:   3.2.1
  - JupyterHub version:   4.0.2
  - Hub pod packages:     See https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/3.2.1/images/hub/requirements.txt

### Followup links

  - Documentation:  https://z2jh.jupyter.org
  - Help forum:     https://discourse.jupyter.org
  - Social chat:    https://gitter.im/jupyterhub/jupyterhub
  - Issue tracking: https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues

### Post-installation checklist

  - Verify that created Pods enter a Running state:

      kubectl --namespace=jupyterhub get pod

    If a pod is stuck with a Pending or ContainerCreating status, diagnose with:

      kubectl --namespace=jupyterhub describe pod <name of pod>

    If a pod keeps restarting, diagnose with:

      kubectl --namespace=jupyterhub logs --previous <name of pod>

  - Verify an external IP is provided for the k8s Service proxy-public.

      kubectl --namespace=jupyterhub get service proxy-public

    If the external ip remains <pending>, diagnose with:

      kubectl --namespace=jupyterhub describe service proxy-public

  - Verify web based access:

    You have not configured a k8s Ingress resource so you need to access the k8s
    Service proxy-public directly.

    If your computer is outside the k8s cluster, you can port-forward traffic to
    the k8s Service proxy-public with kubectl to access it from your
    computer.

      kubectl --namespace=jupyterhub port-forward service/proxy-public 8080:http

    Try insecure HTTP access: http://localhost:8080
```
Чтобы открыть Jupyterhub на localhost 
```
kubectl --namespace=jupyterhub port-forward service/proxy-public 8080:http
```
