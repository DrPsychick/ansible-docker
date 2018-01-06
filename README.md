## DrPsychick.docker-deploy

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

### Example Playbook

    - hosts: dockerhosts
      pre_tasks:
        - include_vars: mycontainers_definition.yml
      roles:
        - { role: DrPsychick.docker-deploy }

## Use case
I use this to test, create, run and update containers in my local environment and on my dedicated server.


## Internals 

Flow:
* copy files/templates to target machine (Dockerfile, env-file, ...)
* pull or build image
* stop and remove existing container
* create and restart container
* cleanup
* setup aliases to maintain containers easily

### License

GPLv3

