# AltSchool_K3s_WebApps
>This is a project implementing DDD with .Net and node on top of K3s using Rancher

### Reproducing K3s Setup
- Clone This repo to a folder using git clone [**OlMofe WebApps** (`digitalOcean`)](https://github.com/rancher/quickstart)

- Register on Digital ocean and create a personal access key with read and write permission.
See how to create [**here** (`digitalOcean`)](https://docs.digitalocean.com/reference/api/create-personal-access-token/)

- navigate to the DigitalOcean folder containing the Terraform files: `cd k8s_Setup_Terraform/rancher/do`

- Add new reposiroty secrets on Github with below Names
    - `RANCHER_DEFAULT_PASS` - this is the default admin password you want to setup rancher with
    - `DO_ACCESS_KEY` - this is the personal access token you created earlier on digitalOcean.
- Optional: If you don't want  to use Github secrets, you can navigate to `k8s_Setup_Terraform/rancher/do` folder, and rename terraform.tfvars.example to terraform.tfvars; then customize the following variables:

- `do_token` - DigitalOcean access key
`rancher_server_admin_password` - Admin password for created Rancher server

- Optional: Modify optional variables within terraform.tfvars. eg.

    - `do_region` - DigitalOcean region, choose the closest instead of the default (nyc1)
    - prefix - Prefix for all created resources
`droplet_size` - Droplet size used, minimum is s-2vcpu-4gb but s-4vcpu-8gb could be used if within budget

- navigate to the github workflows. If you're in the terraform do folder then: `cd ../../../.github/workflows`
- Edit the Applied variable to indicate if you have setup your environment successfully before or not

```
env:
  Applied: 'false'
```
- If this is not your very first run, you'll need to make a change in the the `k8s_Setup_Terraform/rancher/do` folder.
It may be negligible such as adding a text to the readme.
This (and the Applied variable) is to ensure that you actually intend for the terrafoorm job to be run

- Push your code to the main branch on github and watch github actions run.
- Once it's done, look into the output of the `Terraform Apply` for text that looks like the
below(it is probably at the end of the step)

```
Apply complete! Resources: X added, 0 changed, 0 destroyed.

Outputs:

rancher_node_ip = xx.xx.xx.xx
rancher_server_url = https://rancher.xx.xx.xx.xx.sslip.io
workload_node_ip = yy.yy.yy.yy
```

- Paste the `rancher_server_url` from the output above into the browser. Log in when prompted (default username is `admin`, use the password set in `rancher_server_admin_password` earlier).


- To pull down the environment, set DestroyEnv variable to true, make any kind of change in the `k8s_Setup_Terraform/rancher/do` folder and push to min. (Be careful of course)
```
env:
 DestroyEnv: 'true'
```
- Dont forget to update the applied variable

```
env:
  Applied: 'true'
```

---
# Setup up kubeconfig on local machine
>Kubeconfig allows us to access kubernetes APIs and manage our cluster though a CLI
Of course, with rancher, the mature UI can do many of the basic things desired.

- install kubectl on local device (preferably WSL2/linux but mac and windows are supported by kubectl also)
- Go to rancher UI which may look something like this: https://rancher.xx.xx.xx.xx.sslip.io
- naivagte to Cluster Management inside Global apps in the side bar
- click on the options(right - 3 dots) of the Custom `provider` cluster. Following the earlier steps, The cluster name should be called: `quickstart-do-custom`
- download the kubeconfig and save in default location for kubectl `~/.kube` and save file name as `config`. so the file path will be `~/.kube/config`
- you can also save the kubeconfig in `.kube`. You can view a sample format in `.kube/quickstart-do-custom-example.yaml`

- test your connection by listing the non-local clusters with the below command
assuming your config file is called `quickstart-do-custom.yaml`
```
kubectl config view --kubeconfig .kube/quickstart-do-custom.yaml
```
```
kubectl config get-contexts --kubeconfig .kube/quickstart-do-custom.yaml
```
---
# Setup Socks WebApp Example
- navigate to docks kubernetes folder: 
`cd socks-microservices-demo/deploy/kubernetes`

- You can confirm the current cluster that kubectl is currently executing on:
```
kubectl config current-context 
```
- if you find you're interfacing with the wrong context/cluster, simply list the contexts/clusters and switch to the desired one.
```
kubectl config get-contexts
kubectl config use-context cluster-name 
```

- Now Create the desired namespace in the current cluster (you should still be in the kubernetes folder)
```
kubectl create namespace sock-shop
```
- install the application workloads
```
kubectl apply -f complete-demo.yaml
```

- Once it's done, head over to rancher cluster and launch your application.


---
# Setup Fleet for CD
> Fleet is a container management and deployment engine designed to offer users more control on the local cluster and constant monitoring through GitOps.
Fleet can also manage deployments from git of raw Kubernetes YAML, Helm charts, Kustomize, or any combination of the three. Regardless of the source, all resources are dynamically turned into Helm charts, and Helm is used as the engine to deploy all resources in the cluster

- **install helm locally (Optional).** 
- See guide [**here** (`helm`)](https://helm.sh/docs/intro/install/).**
For linux, the below will work.
You can skip this though, if you intend to run the helm command from withing the cluster's kubectl.
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh
```

- **install Fleet Helm charts**
> **_NOTE:_**  This step may fail if you haven't saved your kubeconfig file  in `~/.kube` folder, Alternatively, just run the command in each cluster's CLI (top right).
```
helm -n cattle-fleet-system upgrade -i --create-namespace --wait \
    fleet-crd https://github.com/rancher/fleet/releases/download/v0.6.0-rc.5/fleet-crd-0.6.0-rc.5.tgz
helm -n cattle-fleet-system upgrade -i --create-namespace --wait \
    fleet https://github.com/rancher/fleet/releases/download/v0.6.0-rc.5/fleet-0.6.0-rc.5.tgz
```
> if that fails from your local machine, try the command from kubectl within the each cluster(eg.`quickstart-do-custom` and `local`)  

- Fleet should be ready to use now for single/multi cluster. You can check the status of the Fleet controller pods by running the below commands.
```
kubectl -n cattle-fleet-system logs -l app=fleet-controller
kubectl -n cattle-fleet-system get pods -l app=fleet-controller
```
- The last command should show something like the below
```
NAME                                READY   STATUS    RESTARTS   AGE
fleet-controller-64f49d756b-n57wq   1/1     Running   0          3m21s
```

- **Setup cluster Group (Optional, for multi-cluster)**
> A cluster group multiple clusters together and manage them identically.
- Navigate to the `.kube` folder.
- Edit the `kube-group.yaml` file. You can add multiple key/values under the `matchLabels` field.
- copy all it's content.
- Head over to rancher UI - **GLOBAL APPS** -> **Continuous Delivery**
- Click on the drop down in the upper right and change option to fleet-default.
- Inside the page, Click on Cluster Group ->  click on create (right) -> `Select Edit as YAML`(bottom right).
- paste the content of the kube-group.yaml in the field and click Create.
> You can also create using the form UI.

- Click on the `Clusters` menu (left) 
- Select all the cluster you want to add to the group(using the left checkbox).
1 Select `Change workspace`
- Select `Add/Set Label`
- Set the key/value pair to match the newly created group (eg. `env=test`)
- Select Apply.
> If the cluster or group is stuck in `Wait-Applied` state, simpply slect the `Force Update` option.

- **Add Git Repo**
naviage to kubeconfig folder: `cd .kube`
- Edit the `socks-fleet-repo.yaml` file 
- Set the `repo:` field to your own repo.
- set the `paths:` field to your relevant manifests path(s) of the workloads you want to deploy.
- edit the cluster filters to match with your desired cluster or cluster group.
> Feel free to comment/uncomment any of the cluster selectors. There must be at least one option that can be used to identify the cluster or cluster group.
- Copy the contents of the file.
- Go to Rancher UI (still in `Continuous Delivery` | `fleet-default`)
- Select `Git Repos` -> `Add Repository` -> `Edit As YAML`
- paste the `socks-fleet-repo.yaml` content and select `Create`

> you must have already pushed this repo to github or whatever repo that is to be referenced must be available online.
```
cat > fleet-examples.yaml << "EOF"
#copy from here
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: single-cluster-cd-test
  annotations:
    field.cattle.io/description: Single Cluster GitRepo  Test
#    key: string
  namespace: fleet-local #option fleet-default is available but not advisable
#  labels:
#    key: string
spec:
  branch: master
  paths:
    - single-cluster/helm
#    - string
  repo: https://github.com/rancher/fleet-examples.git
  targets:
    - clusterSelector:
        matchExpressions:
          - key: provider.cattle.io
            operator: NotIn
            values:
              - harvester
  #up until here
EOF
```

- push changes to github
- still inside the `.kube` folder apply GitRepo config:
`kubectl apply -f fleet-examples.yaml --kubeconfig quickstart-do-custom.yaml`

> **_NOTE:_** If you get any of the below error, then you can simply apply the yaml from Rancher Management UI. Simply go to `Continuous Delivery` inside **`GLOBAL APPS`** and add git repo.

> `error: unable to recognize "fleet-examples.yaml": no matches for kind "GitRepo" in version "fleet.cattle.io/v1alpha1"`
> `Error from server (NotFound): error when creating "fleet-examples.yaml": namespaces "fleet-default" not found`

> You can also fill the specs in a form like manner.

- Check the status of what the fleet is doing
```
kubectl -n fleet-local get fleet
```


# Setup Socks shop Example
*To account for resource constraint, I have applied resource constraint on containers that previously did not have it. see below:
```
resources:
    limits:
    cpu: 300m
    memory: 1000Mi
    requests:
    cpu: 100m
    memory: 300Mi
```

