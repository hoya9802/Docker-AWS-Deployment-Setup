# Deployment


## Server Setup

### Creating an SSH Deploy Key

To create a new SSH key which can be used as the deploy key, run the command below:

```sh
ssh-keygen -t ed25519 -b 4096
```

Note: This will create a new `ed25519` key, which is the recommended key for GitHub.

To display the public key, run:

```sh
cat ~/.ssh/id_ed25519.pub
```


### Install and Configure Depdencies

Use the below commands to configure the EC2 virtual machine running Amazon Linux 2 & Amazon Linux 2023.

Install Git:

```sh
sudo yum install git -y
```

Install Docker, make it auto start and give `ec2-user` permissions to use it:

```sh
sudo amazon-linux-extras install docker -y # AL2
sudo dnf install docker -y # AL2023

sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo usermod -aG docker ec2-user
```

Note: After running the above, you need to logout by typing `exit` and re-connect to the server in order for the permissions to come into effect.

Install Docker Compose:

you can check various release version of docker compose [here](https://github.com/docker/compose).

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```


## Running Docker Service


### Cloning Code

Use Git to clone your project:

```sh
git clone <project ssh url>
```

Note: Ensure you create an `.env` file before starting the service.


### Running Service

To start the service, run:

```sh
docker-compose -f docker-compose-deploy.yml up -d
```

To generate the initial SSL certificates using Certbot, run the command below. This step is required before starting the server with HTTPS support:

```sh
docker-compose -f docker-compose-deploy.yml run --rm certbot /opt/certify-init.sh
docker-compose -f docker-compose-deploy.yml down
docker-compose -f docker-compose-deploy.yml up -d
```

### Stopping Service

To stop the service, run:

```sh
docker-compose -f docker-compose-deploy.yml down
```

To stop service and **remove all data**, run:

```sh
docker-compose -f docker-compose-deploy.yml down --volumes
```


### Viewing Logs

To view container logs, run:

```sh
docker-compose -f docker-compose-deploy.yml logs
```

Add the `-f` to the end of the command to follow the log output as they come in.


### Updating App

If you push new versions, pull new changes to the server by running the following command:

```
git pull origin
```

Then, re-build the `app` image so it includes the latest code by running:

```sh
docker-compose -f docker-compose-deploy.yml build app
```

To apply the update, run:

```sh
docker-compose -f docker-compose-deploy.yml up --no-deps -d app
```

The `--no-deps -d` ensures that the dependant services (such as `proxy`) do not restart.
