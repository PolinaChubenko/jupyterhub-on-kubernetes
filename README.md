# jupyterhub-on-kubernetes

1. Installing helm https://helm.sh/docs/intro/install/

```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

2. Install kubectl binary with curl on Linux https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

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


3. Установка Minikube для локального подъема kubernetes https://kubernetes.io/ru/docs/tasks/tools/install-minikube/
   
// DID NOT WORK FOR ME

// Another solution
Installing kind https://kind.sigs.k8s.io/docs/user/quick-start/

Creating a Kubernetes cluster:
`kind create cluster`


4. Running Jupyterhub in Kubernetes https://citizix.com/how-to-run-jupyterhub-in-kubernetes/

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
