# Lab: Read State from Terraform Enterprise Workspace

Duration: 15 minutes

This lab demonstrates how to read state from Terraform Enterprise.

- Task: Create a Terraform config that reads from the state from a different workspace Terraform Cloud

## Prerequisites

For this lab, we'll assume that you've installed [Terraform](https://www.terraform.io/downloads.html) and that you have [signed up](https://app.terraform.io/signup/account) for a Terraform Cloud account and have completed the `terraform-cloud` lab.

## Task: Create a Terraform config that reads from the state on Terraform Cloud

Now that we have our state stored in Terraform Cloud in our `workspace` workspace, we will create another project, configuration, and workspace to read from it.

### Step 1.3.1

Start by creating a new directory called `read-state` with a `main.tf` file:

```shell
mkdir -p /workstation/terraform/read_state && cd $_
```

```shell
touch main.tf
```

### Step 1.3.2

Place the following into your `main.tf`

```hcl
resource "random_id" "random" {
  keepers = {
    uuid = uuid()
  }

  byte_length = 8
}
```

### Step 1.3.3

In order to read from our workspace in Terraform Enterprise, we will need to setup a `terraform_remote_state` data source. Data sources are used to retrieve read-only data from sources outside of our project. It supports several cloud providers, but we'll be using `remote` as the `backend`.

```hcl
# read_state/main.tf
data "terraform_remote_state" "write_state" {
  backend = "remote"

  config = {
    hostname = "app.terraform.io"
    organization = "<ORGANIZATION NAME>"

    workspaces = {
      name = "<WORKSPACE_NAME>"
    }
  }
}
```

### Step 1.3.4

Now that we have access to our remote `server-build` workspace, we can retrieve the `public_ip` output contained within it. We'll also output `random` which we created in this configuration, confirming that they are distinct.

```hcl
# read_state/main.tf
output "random" {
  value = random_id.random.hex
}

output "remote_state_server_public_ip" {
  value = data.terraform_remote_state.write_state.outputs.public_ip
}
```

### Step 1.3.5

To verify that we have successfully retrieved the state from out `server-build` workspace, we can run our configuration and validate our outputs.

Run `init` again to install the necessary supporting files.

```shell
terraform init
```

Run `apply` to see the output.

```shell
terraform apply -auto-approve
```

```
Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

random = "c0ae46c0a3d596ba"
remote_state_server_public_ip = [
  [
    "3.89.6.38",
    "34.235.120.118",
  ],
  [
    "18.234.123.101",
    "18.232.106.115",
  ],
]
```

It worked! You've now successfully stored your states remotely and read from those remote states.