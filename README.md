# Mikado

## Intro

Mikado helps managing your AWS infrastructure for Wordpress sites by defining an out-of-box, highly available, easy-to-deploy setup.

Our goals are:
- Provide an oversimplified but flexible and resilient one-click Wordpress deployment
- Create a widely used standardized Wordpress infrastructure
- Implement performance, security and infrastructure best practices out of box
- Have automated, auditable, and idempotent configuration


## Overview

Mikado provides a fully automated way to deploy and maintain your infrastructure built with [Terraform](https://terraform.io/) and [Packer](https://packer.io/) + [Ansible](https://www.ansible.com/) with the following services integrated optionally:

- [Fastly](https://fastly.com/) - CDN
- [Statuscake](https://statuscake.com/) - external monitoring
- [Datadog](http://datadog.com/) - server monitoring & AWS resource monitoring
- [Loggly](https://loggly.com/) - remote log collection
- [Newrelic](https://newrelic.com/) - application monitoring

## Infrastructure overview

![Mikado overview](https://github.com/dominis/mikado/blob/master/resources/mikado-infra.png)

- Mikado will create it's own VPC with public and private subnets in all the available Availability Zones in the selected region - providing a geo-redundant highly-available setup
- The Wordpress site will be deployed to an Multi-AZ Auto scaling group with a set of pre-defined but fine tunable up/down scaling rules
- Uploaded assets are stored on an EFS drive
- A Multi-AZ RDS cluster is used in the database layer
- Route53 used to manage DNS for the site
- Optionally you can deploy a Fastly service for your site to cache all your requests.

## Quick start

### Building your base AWS infra

Mikado provides a Vagrant instance for local development with all the dependencies installed.

Get the latest version of mikado:
```
git clone http://github.com/dominis/mikado
cd mikado
```

Start the vagrant instance, for this you need to [install vagrant](https://www.vagrantup.com/docs/installation/) first:
```
vagrant up
vagrant ssh
cd mikado
```

You need to create your env file for your credentials:
```
cp env.mk.template env.mk
vim env.mk
```

Once you done you can run terraform to build the base infrastructure:
```
make apply
```
This will create a VPC in the selected region with public and private subnets in all available Availability Zones. This is needed for the next step.

At this point you will be able to create the image for your application servers. This step will start a new EC2 instance in one of your subnets and will provision it by using Ansible. If the process is successful an [AMI](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) will be created. This will be the base for your servers in the Auto Scaling Groups.
```
make build-ami
make deploy-ami
```

If you make it this far you can configure your Wordpress setup. Check out the [examples](https://github.com/dominis/mikado/tree/master/examples). 
```
cp examples/basic-no-fastly.tf terraform/wpexample.com.tf
make apply
```

### Deploying your website

Mikado has a very simple automated deploy workflow based on git and branches.

You need to set the `site_repo` variable in the `env.mk` file in the following format: `https://YOUR_GITHUB_OAUTH_TOKEN:x-oauth-basic@github.com/YOUR_GITHUB_USER/wordpress.example.com.git`

[More info on the token creation](https://help.github.com/articles/creating-an-access-token-for-command-line-use/)


Take a look at the [example repository](https://github.com/dominis/wordpress.example.com). The simplest way to start is forking this repo.

#### Important information about the wordpress deploy process:

- `develop` branch will be deployed to the test server
- `production` branch will be deployed to the prod server
- the `wp-contents/uploads` directory should be ignored in the `.gitignore` and shouldn't exists in the repo, a symlink is created pointing to the EFS mount here automatically
- for the test/prod database config check out the [wp-config.php](https://github.com/dominis/wordpress.example.com/blob/develop/wp-config-sample.php#L32-L36)
- [this is the script](https://github.com/dominis/mikado/blob/master/ansible/roles/wordpress/templates/deploy_wordpress.j2) which pulls the changes from git every minute on the instances


## FAQ

- Q: How can I ssh to my instances
- A: Both the test and prod ELB exposes ssh for the IP blocks in the internal SG (TF_VAR_allowed_cidrs env var), so you can simply `ssh ec2-user@origin.domain.com` or `ssh ec2-user@test.domain.com`.

- Q: ???
- A: !!!
