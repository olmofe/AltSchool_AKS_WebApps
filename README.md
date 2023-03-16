# AltSchool_AKS_WebApps
This is a project implementing DDD with .Net and node on top of K8s using AKS

### Reproducing K8s Setup
- Clone This repo to a folder using git clone [**OlMofe WebApps** (`digitalOcean`)](https://github.com/rancher/quickstart)

- navigate to the DigitalOcean folder containing the Terraform files: `cd k8s_Setup_Terraform/rancher/do`

- Edit terraform.tfvars and customize the following variables:

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
  Applied: false
```
- push code to main branch of repo