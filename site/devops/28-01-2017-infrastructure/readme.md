# Quickstart

This quickstart explains how to setup and install the infrastructure needed for the site.

## requirements

1. terraform must be installed (https://www.terraform.io/)
1. packer must be installed (https://www.packer.io)
1. project-manager must be installed (https://github.com/velezj/project-manager.git, only the project-manager/shunt/shunt/ module is needed)

## setup

1. clone the git repository
```sh
git clone https://github.com/velezj/salient-lore.git
```

1. create your personal devops ssh key-pair for creating and maintaining the infrastructure
```
cd salient-lore/site/devops/28-01-2017-infrastructure/
ssh-keygen -t rsa -b 4096 -f <devops-key-name>
```

1. edit the `shunts/variables` file and update the information therein. Specifically look for and change:
   - `aws_profile`
   - `project`
   - `devops_key`
   - `state_s3_bucket`

## Materialize the wanted infrastructure code

1. run the project-manager shunt script to materialize the infrastructure code
```sh
cd salient-lore/site/devops/28-01-2017-infrastructure/
python <project-manger/shunt/shunt/shunt.py> Shuntfile
```

You should now have a `materialized_views` folder sibling to the Shuntfile.

## Create base images for instances

1. Create the instance images using packer
```sh
cd salient-lore/site/devops/28-01-2017-infrastructure/materialized_views/
packer packer-site.json
```

## Start infrastructure

1. Apply the terraform infrastructure
```sh
cd salient-lore/site/devops/28-01-2017-infrastructure/materialized_views/
terraform init
terraform apply
```

1. Alias commonly used commands for your new infrastructure
```sh
cd salient-lore/site/devops/28-01-2017-infrastructure/materialized_views/
source setup-env.sh
```

## Provision the infrastructure

1. ssh into the site instance
```sh
ssh-into-site
```

1. on site machine, apply salt provisioning
```sh
sudo salt '*' state.highstate
```
