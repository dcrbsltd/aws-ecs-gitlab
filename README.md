# aws-ecs-gitlab
[![Build Status](https://travis-ci.org/dcrbsltd/aws-ecs-gitlab.svg?branch=master)](https://travis-ci.org/dcrbsltd/aws-ecs-gitlab)

An ECS RDS backed Gitlab application, orchestrated via CloudFormation (CF)

## Prerequisites
The scripts to run this CF template require

  * cURL
  * awscli

## Getting started
To create the stack in Amazon, copy the environment.variables.example to environment.variables, thusly

```
  cp environment.variables.example environment.variables
```

fill in the relavent details 
```
  # AWS Keys and Region
  export AWS_ACCESS_KEY_ID=
  export AWS_SECRET_ACCESS_KEY=
  export AWS_DEFAULT_REGION=

  # DNS Domain name which is also used as the Stack name
  export HOST=gitlab
  export DNS_DOMAIN=
  # A stack name is required, which is created from the host and domain with dashes subsituted for periods
  export STACK_NAME=$HOST-`echo $DNS_DOMAIN | sed -e 's/\./\-/g'`
  
  # DB password and LoadBalancer SSL certificate
  export DB_PASSWORD=
  export SSL_CERTIFICATE_ID=

  # Notification Topics for the ASG and CF events
  # Note that you could always set them to the same topic
  export NOTIFICATION_TOPIC_ARN=
  export CF_NOTIFICATION_TOPIC_ARN=
```
and and run the command
```
  make install
```
or simply ```make```.

## Known Issues

### HA vs Backup/restore strategy
Currently this runs on an instance in an AutoScaling Group. Although this is good practice for [the Cloud](http://cloudsarelies.com/), it could be argued that a simple EC2 instance would suffice.

#### Pros:
* Its highly available and load-balanced

#### Cons:
* It requires a complicated backup/restore strategy
* Only one instance can be used due to the filesystem

### Nginx and SSL
Currently the Amazon load-balancer ONLY listens on port 443, proxying the HTTPS requests to port 80 on the instance. 

A more user-friendly configuration where Nginx redirects/rewrites (I forget which) requests from HTTP -> HTTPS can be configured. However the SSL cert and key need to be installed on the instance - which is a bit messy from an automation perspective.

### Travis-ci
Travis has a timeout on jobs which don=t output to logs after 10 minutes, hence the verbose "STACK_STATUS" output.

## Todos
* Create a script that backs up the Gitlab database to an s3 bucket ```gitlab-rake gitlab:backup:create | aws s3 cp...``` for example.
* Investigate Amazon Elastic file system https://aws.amazon.com/efs/ (would solve the AutoScaling argument).
* Play with Nginx config to get redirects working without messy automation.
* Find a way to make the create_stack function generic by passing the parameters as an argument.
* SMTP settings.
* The stack name is based on the host and DNS domain you may want to append a build ID to this - for blue/green deployments etc.

## References
* https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/raketasks/backup_restore.md
* http://doc.gitlab.com/omnibus/docker/
* https://hub.docker.com/r/gitlab/gitlab-ce/
* https://aws.amazon.com/efs/