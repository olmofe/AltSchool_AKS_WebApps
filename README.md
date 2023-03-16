# AltSchool_AKS_WebApps
This is a project implementing DDD with .Net and node on top of K8s using AKS

### Reproducing K8s Setup
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

