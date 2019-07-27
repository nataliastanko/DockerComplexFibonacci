# Docker Complex Fibonacci

#### Project Development Workflow

## Docker multi-container deployment

### Step 1. Config

#### Docker config

We want 3 project apps:

* client
* server
* worker

as 3 containers.

Create ```Dockerfile.dev``` file in each of the app folders.

Build each app in dev env to test it:

    docker build -f Dockerfile.dev .
    docker run ID

Create ```docker-compose.yml``` file in the project root DIR.

##### Environment variables

Setup environment variables with docker.

Environment variables are created in the runtime, not stored in an docker image.

#### Port mapping

Create ```default.conf``` which adds configuration rules to Nginx.

#### Websocket connection

Allow WebSocket connections in nginx conf.

### Step 2. Test and deploy with Travis-CI and Elastic Beanstalk AWS

#### Travis-CI

Create production ```Dockerfile``` file.

Create ```.travis.yml``` file.

Push your docker images to your docker hub repos.

#### Elastic Beanstalk AWS

Create new AWS Elastic Beanstalk instance via AWS dashboard with single container deployment (choose ```multi-docker``` in settings.)

Create ```Dockerrun.aws.json```.

Pull your docker hub images with Elastic Beanstalk AWS in ```Dockerrun.aws.json```.

```Dockerfile/docker-compose.yml``` stores the info how to build an image from services.

```Dockerrun.aws.json``` stores container/tasks [definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions) - instructions on how to run a single container (instance of an image).

#### Databases inside of container

###### Get your own DB running in a container in prod

Example: in digitalocean.com

or

###### Use external data provider services.

##### Use external data provider services

###### VPC - Virtual Private Cloud

You get it (one default) in each of very specific different regions in EB AWS.

In dev env we had Redis and Postgres containers.
In prod env we are going to use:

* AWS ElastiCache instead of Redis
* AWS Relational Database Service (RDS)

in Elastic Beanstalk (EB) instance.

###### Why AWS ElastiCache and AWS RDS

* Super easy to scale
* Built in logging + maintenance
* Probably better security than what we can do
* Easier to migrate off of EB with

###### Why AWS ElastiCache

* Automatically creates and maintains Redis instances for you, you dont have to be a Redis specialist

###### Why AWS RDS

* Automatically creates and maintains Postgres instances for you
* Automated backups and rollbacks

###### RDS

Configure in the same region where EB has been created.

Create database in Amazon RDS, type PostgreSQL.
Fill in:

* DB instance identifier
* Master username
* Master password

Key names need to be equal to your ```docker-compose.yml``` ```api.environment``` config.
Values can be different than set up in ```docker-compose.yml```.

###### ElastiCache

Create Redis database cluster in Amazon ElastiCache (Cluster Mode disabled).
Change Node type to t2 cache.t2.micro (low performance, the cheapest) and Number of replicas to 0. Check subnets. Save with default VPC region number.

###### Security Group

To get services (EB, RDS, ElastiCache) connect with each other we need to create a security group (firewall rules) in AWS, which is: allow any traffic from any other AWS service that has this security group.

Create a new Security Group from VPC service dashboard. Save with default VPC region number. Edit Inbound Rule for this group. Rule type: Custom TCP Rule, Port Range 5432-6379, Source Custom - just created security Group ID.

Now apply the security group ID to each of 3 services: EB, RDS, ElastiCache:

* In Redis AWS:Modify:VPC Security Groups.
* In Amazon RDS:AWS Modify:Network & Security:Security group:Apply immediately.
* In Elastic Beanstalk Service:Configuration:instances:Modify:EC2 security groups

###### Environment variables

We already set them up for Redis and Postgres in AWS services.
Go to Elastic Beanstalk Service:Configuration:Software:Modify:Environment properties. These values are not hidden... (wtf).

We add key names with **URL endpoints** values:

* REDIS_HOST - Open ElastiCache in a new tab:Redis:Description:Primary Endpoint, use this (without a port)
* REDIS_PORT - the port from above
* PGHOST - Open RDS AWS in a new tab:Connect(ivity):Endpoint, use this
* PGPORT - Open RDS AWS in a new tab:Connect(ivity):Port, use this
* PGDATABASE - use values set in [RDS section](#rds)
* PGPASSWORD - use values set in [RDS section](#rds)
* PGUSER - use values set in [RDS section](#rds)

#### IAM AWS keys for deployment

The only file that counts here to be send ver to EB is ```docker-composer.yml```.

Specify deploy section in ```.travis.yml``` with

* ```provider: elasticbeanstalk```
* region - you will find region in your aws domain subdomain
* app - it is the Elastic Beanstalk instance name.
* env - it is the Elastic Beanstalk instance env name, ends with ```-env```
* bucket_name - it is Amazon S3 bucket, go to AWS S3, find bucker name
* bucket_path - same as app

Generate ```$AWS_ACCESS_KEY``` and ```AWS_SECRET_KEY``` on IAM AWS dashboard:Users:Add User:Check Programmatic access:Permissions:Attach existing policies directly:Filter beanstalk:You can check everything or Provides full access to AWS Elastic Beanstalk:Create user without a permissions boundary:No tags:Create User

Copy ```Access key ID``` and ```Secret access key```.

##### Update ```.travis.yml``` file to deploy over to EB:

Generate ```access_key_id``` and ```secret_access_key``` environment variables on Travis-CI dashboard:[project]:More options:Settings:Environment Variables

Create ```AWS_ACCESS_KEY``` key with ```Access key ID``` value.

Create ```AWS_SECRET_KEY``` key with ```Secret access key``` value.

DO NOT DISPLAY VALUE IN BUILD LOG.

#### If health status it not OK

* Check logs in EB

#### Re-deploying application

#### Cleaning Up AWS Resources

to not get charged.

We need to clean:

* Elastic Beanstalk - AWS Elastic Beanstalk dashboard:[project]:Actions:Delete application
* ElastiCache - AWS ElastiCache dashboard:Redis:[project]:Delete
* RDS - AWS RDS dashboard:[project]:Modify:Deletion protection:Uncheck, then AWS RDS dashboard:[project]:Actions:Delete

***Security groups*** do not have to be deleted, but here is the way:

AWS VPC dashboard:Security Groups:check ALL groups connected to your app (leave the default):Actions:Delete

If not successful, try in a few minutes again.

If still not successful, remove Security Group Inbound Rules, then attempt to delete Security Group.

***IAM*** dashboard:Users:Delete

***

***

***

* Based on [udemy course](https://www.udemy.com/docker-and-kubernetes-the-complete-guide/).
