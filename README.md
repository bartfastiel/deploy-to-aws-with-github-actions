# Deploy spring boot application to docker on EC2

## running

1. Create EC2 instance on AWS
   * (keep default:) Amazon Linux 2023 AMI
   * (keep default:) t2.micro or similar is a good start
   * Add security group to allow inbound traffic for SSH, HTTP, HTTPS
2. Install docker
   * ssh to EC2 instance
   * `sudo yum install docker --assumeyes --quiet`
   * `sudo service docker start`
   * `sudo usermod --append --groups docker ec2-user`
   * `newgrp docker`
   * `docker network create capstones`
   * `docker run --pull=always --name app-alpha --network capstones --detach bartfastiel/deploy-to-aws-with-github-actions:latest`
3. Test
    * Open browser and go to EC2 instance public IP address (double check, that you use http:// and not https://)

### to restart

* ssh to EC2 instance
* `docker stop app`
* `docker rm app`
* `docker run --pull=always --name app --network capstones --detach bartfastiel/deploy-to-aws-with-github-actions:latest`

### reverse proxy

* `docker run --pull=always --publish 80:80 --name nginx --network capstones --detach bartfastiel/deploy-to-aws-with-github-actions-nginx:latest`

## building

### app

* in `backend` folder
* `mvn clean install`
* in project root folder
* `docker buildx build --platform linux/amd64 --push --tag bartfastiel/deploy-to-aws-with-github-actions:latest .

### nginx

* in `nginx` folder
* `docker buildx build --platform linux/amd64 --push --tag bartfastiel/deploy-to-aws-with-github-actions-nginx .`
