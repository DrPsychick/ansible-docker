## DrPsychick.ansible-docker

[![Build Status](https://travis-ci.org/DrPsychick/ansible-docker.svg?branch=master)](https://travis-ci.org/DrPsychick/ansible-docker) 
[![license](https://img.shields.io/github/license/drpsychick/ansible-docker.svg)](https://github.com/drpsychick/ansible-docker/blob/master/LICENSE) 
[![GitHub stars](https://img.shields.io/github/stars/drpsychick/ansible-docker.svg)](https://github.com/drpsychick/ansible-docker) 
[![Contributors](https://img.shields.io/github/contributors/drpsychick/ansible-docker.svg)](https://github.com/drpsychick/ansible-docker/graphs/contributors)

[![GitHub issues](https://img.shields.io/github/issues/drpsychick/ansible-docker.svg)](https://github.com/drpsychick/ansible-docker/issues) 
[![GitHub closed issues](https://img.shields.io/github/issues-closed/drpsychick/ansible-docker.svg)](https://github.com/drpsychick/ansible-docker/issues?q=is%3Aissue+is%3Aclosed) 
[![GitHub pull requests](https://img.shields.io/github/issues-pr/drpsychick/ansible-docker.svg)](https://github.com/drpsychick/ansible-docker/pulls) 
[![GitHub closed pull requests](https://img.shields.io/github/issues-pr-closed/drpsychick/ansible-docker.svg)](https://github.com/drpsychick/ansible-docker/pulls?q=is%3Apr+is%3Aclosed)
<!--- 
[![GitHub last commit (branch)](https://img.shields.io/github/last-commit/drpsychick/ansible-docker/master.svg)](https://github.com/drpsychick/ansible-docker) 
--->


Purpose:
* manage a simple docker environment with ease and flexibility
* use basic linux and docker commands
* automate build, create, restart of your containers
* rollout only required changes
* use it in your NON-PRODUCTION environment, at home or in your test lab

NOT included:
* random container name
* starting container multiple times/load balancing/high availability

### Requirements
* install docker (I use geerlingguy.docker to install docker on the hosts)

### Role variables
See `defaults/main.yml`. Include your own definition of 'containers' dict.

minimum definition:
```
containers:
  mycontainer:
    hosts: ["all"]
    name: "mytestcontainer"
    image: "cogniteev/echo"
    directory: "mytestcontainer"
    pull: yes
```

Special variables to limit your play to what you need. Will affect all instance of the role.
```
container_only=mytestcontainer
container_forcebuild=yes
container_forcecreate=yes
container_forcerestart=yes
```

### Example Playbook

    - hosts: dockerhosts
      pre_tasks:
        - include_vars: mycontainers_definition.yml
      roles:
        - { role: DrPsychick.ansible-docker }

## Use case
I use this to test, create, run and update containers in my local environment and on my dedicated server.

## Examples
Setup your own role which defines the docker containers and includes required files and templates. With this you can keep configuration and source files separate from the imported role and control different scenarios.

Directory structure:
```
container-definition
|- files : files for containers in container.name subdir
   |- mytestcontainer
      |- testfile
|- templates : templates for containers in container.name subdir
   |- mytestcontainer
      |- mytestcontainer.env
      |- Dockerfile
|- tasks : if you want to selectively include variables or set specific variables
|- vars : definition of containers
```

Variables:
```
# sets the source directory for files and templates to the current role path
# cannot use "role_path" as it gets reevaluated in "ansible-docker" role somehow
container_default_sourcedir: '{{ playbook_dir + "/roles/containerdefinition" }}'

containers: 
  mytestcontainer:
    name: "mytestcontainer2"
    build: yes
    templates:
      dockerfile: { name: "Dockerfile", mode: "0644" }
      envfile: { name: "mytestcontainer.env", mode: "0644" }

```

Playbook:
```
- name: Setup docker containers
  hosts: all
  roles:
    - { role: containerdefinition }
    - { role: DrPsychick.ansible-docker }
```

## Internals 

Flow:
* copy files/templates to target machine (Dockerfile, env-file, ...)
* pull or build image
* stop and remove existing container
* create and restart container
* cleanup
* setup aliases to maintain containers easily

### Testing
* create `ansible.cfg` file with content:
```
[defaults]
roles_path = ../
```
* run lint + tests (see `.travis.yml`):
  * `yamllint` - need to somehow pass ansible-galaxy rules...
  * `ansible-lint ./`
  * `ansible-playbook -i tests/inventory tests/test.yml --connection=local --become`
  * `ansible-playbook -i tests/inventory tests/test_cleanup.yml --connection=local --become`

### License

GPLv3
