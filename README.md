ALKS Provider for Terraform
=========

[![Build Status](https://travis-ci.org/Cox-Automotive/terraform-provider-alks.svg?branch=master)](https://travis-ci.org/Cox-Automotive/terraform-provider-alks)

This module is used for creating IAM Roles via the ALKS API.

## Pre-Requisites

* An ALKS Admin or IAMAdmin STS session is needed. PowerUser access is not sufficient to create IAM roles.
    * This tool is best used by users with the `Admin` role
    * If you have an `IAMAdmin/LabAdmin` role, you'll be able to create roles and attach policies, but you won't be able to create other infrastructure.
* Works with [Terraform](https://www.terraform.io/) version 0.8 or newer.

## Installation

* Download and install [Terraform](https://www.terraform.io/intro/getting-started/install.html)

```
wget https://releases.hashicorp.com/terraform/0.9.0/terraform_0.9.0_darwin_amd64.zip && unzip terraform*.zip
```

* Download ALKS Provider binary for your platform from [Releases](https://github.com/Cox-Automotive/terraform-provider-alks/releases)

```
curl -L https://github.com/Cox-Automotive/terraform-provider-alks/releases/download/0.9.0/terraform-provider-alks-darwin-amd64.tar.gz | tar zxv
```

* Configure Terraform to find this plugin by creating `~/.terraformrc` on *nix and `%APPDATA%/terraform.rc` for Windows.

```
providers {
    alks = "/path/to/terraform-provider-alks"
}
```

Note: Provide full path to the location of the plugin, unless terraform-provider-alks in in your path


## Usage

1. Export a valid ALKS IAM session to your environment variables - be sure to use either `Admin` or `IAMAdmin` role. The ALKS provider is only responsible for creating the initial role.

`eval $(alks sessions open -i -a "######/ALKSAdmin - sdgsgasf" -r "Admin")`.

If you create a session using the `Admin` role the STS credentials can be shared between the AWS and ALKS providers. If you use an `IAMAdmin` role then you will need to create a `PowerUser` session for the ALKS provider as `IAMAdmin` is limited to IAM-only resources.

2. Edit your terraform scripts to configure the alks provider and create necessary ALKS resources.

3. Run `terraform plan`, `terraform apply` or other commands, as needed and roles can be generated via Terraform.

### Provider Configuration

#### `alks`

```
provider "alks" {
    url        = "<ALKS_URL>"
    account    = "<ALKS_ACCOUNT>"
    access_key = "<ALKS_ACCESS_KEY_ID>"
    secret_key = "<ALKS_SECRET_ACCESS_KEY>"
    token      = "<ALKS_SESSION_TOKEN>""
}
```

Provider Options:
* `url` - (Required) The URL to your ALKS server. Also read from `ENV.ALKS_URL`
* `account` - (Required) The ALKS account to use. Also read from `ENV.ALKS_ACCOUNT`
* `access_key` - (Required) The access key from a valid STS session.  Also read from `ENV.ALKS_ACCESS_KEY_ID`.
* `secret_key` - (Required) The secret key from a valid STS session.  Also read from `ENV.ALKS_SECRET_ACCESS_KEY`.
* `token` - (Required) The session token from a valid STS session.  Also read from `ENV.ALKS_SESSION_TOKEN`.

You can see all available accounts by running: `alks developer accounts`.

### Resource Configuration

#### `alks_iamrole`

```
resource "alks_iamrole" "test_role" {
    name                     = "My_Test_Role"
    type                     = "Amazon EC2"
    include_default_policies = false
}
```

Value                             | Type     | Forces New | Value Type | Description
--------------------------------- | -------- | ---------- | ---------- | -----------
`name`                           | Required | yes        | string     | The name of the IAM role to create. This parameter allows a string of characters consisting of upper and lowercase alphanumeric characters with no spaces. You can also include any of the following characters: =,.@-. Role names are not distinguished by case.
`type`                           | Required | yes        | string     | The role type to use. [Available Roles](https://gist.github.com/brianantonelli/5769deff6fd8f3ff30e40b844f0b1fb4)
`include_default_policies`                           | Required | yes        | bool     | Whether or not the default managed policies should be attached to the role.
`role_added_to_ip`                           | Computed | n/a        | bool     | Indicates whether or not an instance profile role was created.
`arn`                           | Computed | n/a        | string     | Provides the ARN of the role that was created.
`ip_arn`                           | Computed | n/a        | string     | If `role_added_to_ip` was `true` this will provide the ARN of the instance profile role.

## Example

Check out `test.tf` for an very basic Terraform script which:

1. Creates an AWS provider and ALKS provider
2. Creates an IAM Role via the ALKS provider
3. Attaches a policy to the created role using the AWS provider

This example is meant to show how you would combine a typical AWS Terraform script with our custom provider in order to automate the creation of IAM roles.

## Building from Source

If you wish to work on the ALKS provider, you'll first need [Go](http://www.golang.org/) installed on your machine (version 1.8+ is required).

For local dev first make sure Go is properly installed, including setting up a [GOPATH](http://golang.org/doc/code.html#GOPATH). You will also need to add `$GOPATH/bin` to your `$PATH`.

Next, using Git, clone this repository into `$GOPATH/src/github.com/Cox-Automotive/terraform-provider-alks`. All the necessary dependencies are either vendored or automatically installed (using [Godep](https://github.com/tools/godep)), so you just need to type `make build test`. This will compile the code and then run the tests. If this exits with exit status 0, then everything is working! Check your `examples` directory for a sample Terraform script and the generated binary.

```
cd "$GOPATH/src/github.com/Cox-Automotive/terraform-provider-alks"
make built test
```

If you add any additional depedencies to the project you'll need to run `godep save` to update `Godeps.json` and `/vendor`.
