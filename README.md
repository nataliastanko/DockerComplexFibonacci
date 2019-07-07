## Docker Complex Fibonacci

### Multi-container Deployment

#### Step 1.

We want 3 apps

* client
* server
* worker

as 3 containers.

Create ```Dockerfile.dev``` file in each of the app filders.

Build each app in dev env to test it:

    docker build -f Dockerfile.dev .
    docker run ID

Create ```docker-compose.yml``` file in the project root DIR.

Setup environment variables with docker.

##### Environment variables

Environment variables are created in the runtime, not stored in an docker image.

##### Port mapping

Create ```default.conf``` which adds configuration rules to Nginx.

***

***

***

* Based on [udemy course](https://www.udemy.com/docker-and-kubernetes-the-complete-guide/).
