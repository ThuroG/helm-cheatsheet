# This cheatsheet is to explain Helm for a Beginner. Based on KodeKloud


# What is Helm
- Is a packaging manager for Kubernetes to template YAMLs
- For Helm a cluster is needed as well as Kubectl
- Helm 3 is the current version
- Helm comes with a CLI to perform on the Kubernetes Cluster
- Whenever you perform an installation or upgrade, Helm creates a Revision which you can use to rollback
- Helm 3 is using a 3-Way strategic Merge Patch where the current chart (1) will be checked among the last revision (2) and if it fits the Live State (3)
- Different Components of Helm
-- CLI: Perform Helm actions
-- Chart: are collection of files and contains all the information about
-- Release: When a chart is applied on the cluster, a release is created for every helm chart. It can contain multiple revisions
-- Revision: Snapshot of the application
-- Public Repository: Similar to Docker hub. Famous is bitnami
-- Metadata: Save information about Helm and saves it as Kubernetes Secret
-- Values.yaml contains the information for every YAML File. It acts as a settings file
- Helm hub is ArtifactHub.io. You find all charts here!
- Helm Charts are an automation tool to get to the desired version
- A Template in a Helm Chart contains a reference to the values file. For Example: ```{{ .Values.replicaCount}}```
- It is important that the Chart.yaml contains the ```apiVersion: v2``` as it is needed by Helm 3
- The appVersion field is for the application version. For example Wordpress version 5.8.1
- the version field is the current version of this HelmChart
- Workflow Helm
1. Search for the Helm Repo in the Hub with ```helm search hub <name>```
2. Add repo then with ```helm repo add bitnami https://charts.bitnami.com/binami```
3. install with a release name with ```helm install my-release bitnami/wordpress```
- Search with repo only works when the repo has been added to HELM
- Lifecycle
-- Install (multiple) releases of the same helm chart
-- Upgrade a specific version with ```helm upgrade nginx-release bitnami/nginx```
- Sometimes helm needs administrative support for upgrade for example install something or use a password
- Helm rollback will ONLY rollback the application and not the data!


# Command
- Install a helm package with ```helm install wordpress```
- Upgrade an already installed helm package with ```helm upgrade wordpress```
- Rollback an already rollout version with ```helm rollback wordpress```
- Uninstall: ```helm uninstall wordpress```
- Instlal helm on linux:  ```sudo snap install helm --classic```
- Install a release of wordpress (my-site) - can be used for different env or setups ```helm install my-site bitnami/wordpress```
- Helpful information ``` helm --help```
- Helpful informationfor specific action ```helm search --help```
- Or subcommand help ```helm repo update --help```
- Search a specific chart (ex: wordpress) ```helm search hub wordpress``` --> IMPORTANT: HUB!!!
- List all charts (installed) with ```helm list```
- Unistall a release ```helm uninstall my-release```
- helm repo is to add repo 
- passing a set option to pass variable in the install command ```helm install --set wordpressBlogName="Helm Tutorials" my-release bitnami/wordpress --set=...```
- Can also be added to a custom-values.yaml and then added as parameter with ```helm install --values custom-values.yaml my-release bitnami/wordpress```
- helm pull will pull the whole chart locally in a compressed form ```helm pull bitnami/wordpress```
- Untar the compressed chart ```helm pull --untar bitnami/wordpress```
- (!) Find the repo url with ```helm search hub argo --list-repo-url```
- use a specific helm chart version (pay attention - this is not the app version!) with ```helm install nginx-release bitnami/nginx --version 7.1.0```
- Display what happened for a specific release with ```helm history nginx-release```
- Rollback to a specific Revision ```helm rollback nginx-release 1```




