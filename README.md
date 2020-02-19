# hut6/docker-cli-utils

Utilities to help with docker deployments

## Prerequisites

```
composer req --dev hut6/docker-cli-utils
composer req --dev hut6/tag-release
composer req --dev deployer/dist
```

You will need to create a /hosts.yml inventory file (as per Deployer https://deployer.org/docs/hosts.html) and list all the hosts used for deployments.

A VERSION file is also required. Version increments can be managed with the tag release script

You will also need a custom recipe, Dockerbuild file, and at least one Docker Compose file for each project. Example files are available at https://github.com/hut6/2020-docker-prod-boilerplate

## Building

```
bin/build master                                   # branch name
bin/build branch_name                              # branch name
bin/build 1.0.0                                    # tag
bin/build 042f86676a45                             # short hash
bin/build 2ba2014b133ad9feda8fc0c5843dd8dcbf2a8c00 # long hash
```

The image name will be determined using the git remote; if the remote is `git@github.com:hut6/docker-cli-utils.git` then the docker image will be called `hut6/docker-cli-utils`.

The command argument will be used as the tag for the Docker image. If building `bin/build branch_name`, then it will be tagged like this `hut6/docker-cli-utils:branch_name`  

Another example would be `bin/build 1.0.0` => `hut6/docker-cli-utils:1.0.0`.

When building the image, the local files are not used, the remote is used; changes need to be pushed to the remote before the image can be built. 

## Pushing

Same as above. `bin/push branch_name` will push `hut6/docker-cli-utils:branch_name` to DockerHub. The image needs to exist locally (ie needs to have been built). 

## Deploying

For each host, specify the Docker Compose file like this: `compose_file: production.yml`

```
bin/deploy branch_name production
```

First argument is the tag (as per build and push process) and second argument the host, as per the hosts.yml file. This will deploy `hut6/docker-cli-utils:branch_name` on the `production` host.

The image needs to exist on DockerHub. 

## Secrets Management

You'll need the 1Password CLI client `op` for push/pull. 

Install it from https://support.1password.com/command-line/

#### Required files & directories

You'll need to create a `/secrets` directory which will contain a `environment.env` file for each of your deployment environments, eg `production.env`, or `staging.env`

Use the `/hosts.yml` inventory file to specify which entry in 1Password should be used for each hosts, using the `secrets_uuid` key.

#### Pushing to 1Password

To push your file from the secrets directory to 1Password, run the following command.

```
bin/secrets push production
```

Replace production with the right host key, as per your hosts.yml file. 

The first time you run this command, it will ask you which vault needs to be used. 

The same command can be used to update the entry in 1password after you make changes to production secrets file.

#### Pulling from 1Password

This will pull the secrets file from 1password, using the UUID specified in hosts.yml, and add it to the secrets directory. 

```
bin/secrets pull production 
```

#### Updating the docker secrets 

This will load the secrets (from the secrets directory) into the docker daemon for the specified host. Containers likely will need restarting, and this command doesn't do this. 

```
bin/secrets load production 
```
