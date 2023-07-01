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
   * `docker network create capstones --subnet=10.0.0.0/16`
   * `docker run --pull=always --name alpha --network capstones --ip 10.0.1.1 --restart always --detach bartfastiel/deploy-to-aws-with-github-actions:latest`
3. Test
    * Open browser and go to EC2 instance public IP address (double check, that you use http:// and not https://)

### to restart

* ssh to EC2 instance
* `docker stop alpha`
* `docker rm alpha`
* `docker run --pull=always --name alpha --network capstones --ip 10.0.1.1 --restart always --detach bartfastiel/deploy-to-aws-with-github-actions:latest`

### reverse proxy

* `docker run --pull=always --publish 80:80 --name nginx --network capstones --ip 10.0.0.2 --restart always --detach bartfastiel/deploy-to-aws-with-github-actions-nginx:latest`

## building

### app

* in `backend` folder
* `mvn clean install`
* in project root folder
* `docker buildx build --platform linux/amd64 --push --tag bartfastiel/deploy-to-aws-with-github-actions:latest .

### nginx

* in `nginx` folder
* `docker buildx build --platform linux/amd64 --push --tag bartfastiel/deploy-to-aws-with-github-actions-nginx .`

# certificate from letsencrypt

* `sudo docker run --env AWS_ACCESS_KEY_ID=<FIXME> --env AWS_SECRET_ACCESS_KEY=<FIMXE> --interactive --tty --rm --name certbot --volume "/etc/letsencrypt:/etc/letsencrypt" --volume "/var/lib/letsencrypt:/var/lib/letsencrypt" certbot/dns-route53 certonly --dns-route53 --domain *.capstone-project.de --domain capstone-project.de`
