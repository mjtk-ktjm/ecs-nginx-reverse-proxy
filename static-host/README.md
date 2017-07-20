## NGINX Static Host local development container

### Use the docker compose to easily run with this config on your local machine.

```
it-15438:ecs-nginx-reverse-proxy kellym1-v$ cd static-host/
it-15438:static-host kellym1-v$ docker-compose up --build
Building nginx
Step 1/3 : FROM nginx
 ---> e4e6d42c70b3
Step 2/3 : COPY nginx.conf /etc/nginx/nginx.conf
 ---> Using cache
 ---> 72973bb551f6
Step 3/3 : COPY /srv /srv
 ---> Using cache
 ---> 46f288c1ba09
Successfully built 46f288c1ba09
Successfully tagged statichost_nginx:latest
```

*Avoid issues by tearing down those resources once you are done with the container*

```
nginx_1  | 172.25.0.1 - - [20/Jul/2017:17:09:29 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/603.3.8 (KHTML, like Gecko) Version/10.1.2 Safari/603.3.8"
^CGracefully stopping... (press Ctrl+C again to force)
Stopping statichost_nginx_1 ... done
it-15438:static-host kellym1-v$ docker-compose down
```

### You can use the following code to run using just the docker command, for extra flexibility:

```
it-15438:static-host kellym1-v$ docker images | grep static
it-15438:static-host kellym1-v$ docker images | grep static-server
it-15438:static-host kellym1-v$ docker build --rm -t static-server .
Sending build context to Docker daemon  8.192kB
Step 1/3 : FROM nginx
 ---> e4e6d42c70b3
Step 2/3 : COPY nginx.conf /etc/nginx/nginx.conf
 ---> Using cache
 ---> b4ad32a8684e
Step 3/3 : COPY /srv /srv
 ---> Using cache
 ---> b7fbdb43361f
Successfully built b7fbdb43361f
Successfully tagged static-server:latest
it-15438:static-host kellym1-v$ docker run -p 80:8000 static-server:latest
172.17.0.1 - - [20/Jul/2017:16:38:46 +0000] "GET / HTTP/1.1" 200 78 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/603.3.8 (KHTML, like Gecko) Version/10.1.2 Safari/603.3.8"
```

## NGINX Static Host Task Definition for ECS

1. Customize the content that you wish to serve, by changing files inside the `/srv` directory.
2. Build your NGINX container:
   ```
   $ docker build -t nginx_sample .
   ```
3. Create an EC2 Container Registry to push your NGINX container to:
   ```
   $ aws ecr create-repository --repository-name nginx
   ```
4. Authenticate with your new repository:
   ```
   $ `aws ecr get-login`
   ```
5. Tag your image and push your NGINX container to the repository. (You must put it your own repository URI from the `repositoryUri` value from step #3)
   ```
   $ docker tag nginx_sample <repo uri>:revision_1
   $ docker push <repo uri>:revision_1
   ```
6. Modify `task-definition.json` with the the image URL you just pushed to your ECR:
   ```
   {
     "containerDefinitions": [
       {
         "name": "nginx",
         "image": "<repo uri>:revision_1",
         "memory": "256",
         "essential": true,
         "portMappings": [
           {
             "containerPort": "8000",
             "protocol": "tcp"
           }
         ]
       }
     ],
     "volumes": [],
     "networkMode": "bridge",
     "placementConstraints": [],
     "family": "nginx"
   }
   ```
7. The above task definition will launch your NGINX container, with the container's port bound to a dynamically chosen port on one of your ECS hosts. To direct web traffic to that dynamic port you will want to [create an ALB and associate it with your ECS service](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-application-load-balancer.html).
