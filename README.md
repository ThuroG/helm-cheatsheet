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
- If you create a helm chart it creates the structure with sample data. You still have to doit by yourself! For example the templates can be removed if you want to use yours
- Lint the helm chart first before youinstall
- Make sure that you have to templatize for example with the Release Name like ```{{ .Release.Name }}-app```
- Other Template names out of the box are
-- Release.Name / Release.Namespace / Release.IsUpgrade / Release.IsInstall / Release.Revision / Release.Service
-- or Chartspecific templating like Chart.Name / Chart.ApiVersion / Chart.Version / Chart.Type / Chart.Keywords / Chart.Home
-- or from K8s itself with Capabilities.KubeVersion / Capabilities.HelmVersion / Capabilities.GitCommit etc.
-- or of course from the Values file
- Named Templates is to reuse the same line and can be added to _helpers.tpl file: https://helm.sh/docs/chart_template_guide/named_templates/
- You may have to use the "." so that the current scope is used (otherwise only the key will be used)
- Instead of template you should use "include" because it is a function (unlike template which is an action) --> include can be used for pipelines
- You may have to use the function "indent" in order to make sure that the correct indention is used for example: ```{{- include "labels" . | indent 2}}``` --> Adds two more intend at the end


# Functions
- INFO: Helps transform an input to an output and can be used for default values
- upper("helm) --> HELM
- Upper Example in templating: ```{{ upper .Values.image.repository}}``` --> Output: image: NGINX
- Quote example: ```{{ quote .Values.image.repository}}``` --> Output: image: "nginx"
- Replace example: ```{{ replace "x" "y" .Values.image.repository }}``` --> Output: image: "nginy"
- Default value: ```{{ default "nginx" .Values.image.repository}}``` Output: image: "nginx"
- Using a pipe can be used for multiple sequential functions like ```{{ .Values.image.repository | upper | quote }}```--> Output: image: "NGINX"

# Conditionals, Blocks, Loops and Ranges
- One can use conditions for example if a value is set then a certain block should be displayed like:
```
    {{- if .Values.orgLabel }}
        labels:
            org: {{ .Values.orgLabel }}
    {{- else if eq .Values.orgLabel "hr" }}
        labels:
            org: human resources
    {{- end }}
```
- If you add a '-' after the second curly brackets, the empty line in the output dissappears. 
- Mit dieser technique können ganze Blocks / Kubernetes Objects generiert werden. Bsp. in values.yaml steht create: true (for service account) und der gesamte ServiceAccount.yaml Block ist in der Condition
- Wiederholung von Hiearchien können mit ```{{- with .Values.app }}``` verkürzt werden
- wenn eine .with Statement zu beginn genutzt wird mit einer true / false condition, kann entschieden werden ob die K8s Objekt generiert wird oder nicht ```{{- with .Values.serviceAccount.create }}```
- Wenn innerhalb dieser Verkürzung aber etwas von Root (also '.') geholt werden muss, verwendet man ``` {{ $.Release.Name }}```
- Eine Liste kann mit ```{{ range .Values.region }} - {{. }} {{ end }}``` mehrfach erstellt werden (derDot ist dabei die Value der liste)

# Chart Hooks
- Can be used to define another action after a certain command. For example create a backup of DB after performing "helm upgrade"
- There are multiple phases which you can use a hook such as pre-upgrade or post-install
- Create a hook by using a shell script and add it as a job in kubernetes
- add the job.yaml to the Helm chart and make sure it contains the annotation "helm.sh/hook": <phase>
- You can have multiple jobs / hooks but when you want to have an order you have to add "helm.sh/hook-weight": "number" to the annotations
- Make sure you use the hook-delete policy otherwise it will run forever
- Documentation: https://helm.sh/docs/topics/charts_hooks/


# Packaging and Signing Charts
- Use command ```helm package ./nginx-chart``` to package it so that it can be uploaded to a chart repository
- Signing it is crucial in order to make sure that no hacker used a malicious data
- Signing with with a private key so that it creates a provenance file
- Create the private key with  ```gpg --quick-generate-key "Arthur Gassmann"``` (for development)
- Upload the public key to an open pgp server
- For production use this command: ```gpg --full-generate-key "Arthur Gassmann"```
- Helm uses an old version of signing and therefore you may need to export the keys to the old format with ```gpg --export-secret-keys >~/.gnupg/secring.gpg```
- Helm package can now be signed with ```helm package --sign --key 'Arthur Gassmann' --keyring ~/.gnupg/secring.gpg ./nginx-chart```
- List the gpg keys: ```gpg --list-keys```
- it creates a SHA512 key and can be checked now with ```sha256sum nginx-chart-0.1.0.tgz```
- Helm can verify the signature with ```helm verify ./nginx-chart-0.1.0.tgz```
- It will throw an error because the publickey is not with it: ```gpg --export 'Arthur Gassmann' > mypublickey```
- Now use this: ```helm verify --keyring ./mypublickey ./gninx-0.1.0.tgz```
- Public user canget the public key from gpg server with ```gpg --recv-keys --keyserver keyserver.ubuntu.com 8D4XXXXX```
- Always use the verify command like ```helm install --verify nginx-chart-0.1.0```
- The package contains of the helm chart zip (tgz), an index.yaml for repository metadata and the provenance file to check the signature. Add it to a folder.
- To create the index.yaml you need to run``` helm repo index nginx-chart-files/ --url https://example.com/charts```
- Upload it on any cloud service or github
- Let the people know your URL and they have to run this: ```helm repo add our-cool-chart https://example-charts.storage.googleapis.com```

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
- Create a helm chart with ```helm create nginx-chart```
- Lint helps to verify if the yaml and structure is correct ```helm lint ./nginx-chart```
- Template helps to check if templating is correctly with ```helm template ./nginx-chart --debug``` Mind that the debug shows the place where it is incorrect
- Dry run the Helm Chart when the error was not catched by lint / template because it is K8s specific (like container instead of container*S*) with ```helm install release-name ./nginx-chart --dry-run``` Note: the dryrun flag does not install it




