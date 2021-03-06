# Note
Do not use this for production, for now!

# Howto use:
## Instance types
**bold** is default  

| Terraform value | Concourse Web     | Concourse Worker  | postgres          |
| --------------- | :---------------: | :---------------: | :---------------: |
| small           | **n1-standard-1** | -                 | **n1-standard-1** |
| medium          | n1-standard-2     | **n1-standard-2** | n1-standard-2     |
| large           | n1-standard-4     | n1-standard-4     | n1-standard-4     |
| xlarge          | n1-standard-8     | n1-standard-8     | n1-standard-8     |
| 2xlarge         | n1-standard-16    | n1-standard-16    | n1-standard-16    |
| 4xlarge         | -                 | n1-standard-32    | n1-standard-32    |
| 10xlarge        | -                 | n1-standard-64    | n1-standard-64    |
| 16xlarge        | -                 | n1-standard-96    | n1-standard-96    |

## Create a master DNS Zone
You'll want to create a master dns zone if you don't have one in GCP.  
The domain deployed will be a subdomain of this master dns zone  
Example:  
master dns zone = `gcp.mydomain.com` (this resource has a name in GCP, use that as the value of `master_dns_zone_name`)  
subdomain that will be created = `myenv.gcp.mydomain.com` (the value of the variable `dns_domain_name`)

## Create a GCP Service account
To use terraform, you'll need the GCP of a service account.  
This service account can be created using :
```sh
gcloud auth login
gcloud config set project <PROJECT_NAME>
gcloud iam service-accounts create <SERVICE_ACCOUNT_NAME>
gcloud iam service-accounts keys create gcp-key.json \
  --iam-account=<SERVICE_ACCOUNT_NAME>@<PROJECT_NAME>.iam.gserviceaccount.com
gcloud projects add-iam-policy-binding <PROJECT_NAME> \
  --member='serviceAccount:<SERVICE_ACCOUNT_NAME>@<PROJECT_NAME>.iam.gserviceaccount.com' \
  --role='roles/editor'
```

## Enable the GCP APIs
Within Google Cloud Platform, enable the following:
  * GCP Compute API [here](https://console.cloud.google.com/apis/api/compute_component)
  * GCP Storage API [here](https://console.cloud.google.com/apis/api/storage_component)
  * GCP SQL API [here](https://console.cloud.google.com/apis/api/sql_component)
  * GCP DNS API [here](https://console.cloud.google.com/apis/api/dns)
  * GCP Cloud Resource Manager API [here](https://console.cloud.google.com/apis/api/cloudresourcemanager.googleapis.com/overview)
  * GCP Storage Interopability [here](https://console.cloud.google.com/storage/settings)

## Download credhub-cli >=1.7.0
You'll need to download credhub-cli >=1.7.0 from the [credhub-cli repo](https://github.com/cloudfoundry-incubator/credhub-cli)
## Deploy
Create a terraform.tfvars file with :
```sh
# Used to prefix every object
env_name = "MyEnv"

# Name of the GCP Project
gcp_project_name = "GCP-Project-Name"

# GCP Region to use
gcp_region = "europe-west1"

# GCP Zones list in the region (up to 3 entries in the list).
# Do not modify the order once created. You can add but not delete entries
# Eg for 3 entries: gcp_zones = ["europe-west1-b", "europe-west1-c", "europe-west1-d"]
gcp_zones = ["europe-west1-b"]

# The master DNS Zone name (not the actual fqdn, but the name of the resource in GCP)
master_dns_zone_name = "my-zone-name"

# Subdomain of the master zone, which will be created. All entries for Concourse, credhub and UAA will be created in this subdomain.
dns_domain_name = "xxx.master-dns-zone.com"

# Must be a /24
bootstrap_subnet = "10.0.0.0/24"

# Can be 0.0.0.0/0 for full access or a list of IPs/subnets for restricted access
source_admin_networks = ["x.y.z.t/w", "1.2.3.4/16"] 

# Optional (default is small)
# concourse_web_vm_type = "small"

# Optional (default is medium)
# concourse_worker_vm_type = "medium"

# Optional (default is 1): Number of Concourse web VMs to deploy
# concourse_web_vm_count = 1

# Optional (default is 1): Number of Concourse workers to deploy
# concourse_worker_vm_count = 1

# Optional (default is false): Debug enabled
# debug = "false"

# Optional (default is 10): Size of the Database persistent disk
# db_persistent_disk_size = "10"

# Optional (default is small): Size of the postgres DB VM
# db_vm_type = "small"

# Optional (default is false): Deploy grafana and influxdb to monitor the solution
# deploy_metrics = "false"
```

Also export your GCP key:
```sh
export TF_VAR_gcp_key=$(cat gcp-key.json)
```

Then simply run `terraform init && terraform apply`

## Destroy
Simply run `terraform destroy`
All deployments will be deleted and all your resources also (Verify that's really the case :) )