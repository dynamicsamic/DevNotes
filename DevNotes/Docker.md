---
Created: 2024-07-17T10:25
---
#### `Install` Docker on LInux
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# To install the latest version, run:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify that the Docker Engine installation is successful by running the hello-world image
sudo docker run hello-world
```
---
#### Docker `Compose` commands
```bash
# Build and run services in a detached mode (not seeing logs)
sudo docker compose up -d --build

# Stop the composed services.
sudo docker compose -p {compose_name} stop
```
---
#### Build and run standalone services
```bash
# Download, build, add to network and run RabbitMQ server in a named container.
sudo docker run -it --rm --name my-rabbitmq --network my-network -p 5672:5672 -p 15672:15672 rabbitmq:3.8-management-alpine

# Download, build, add to network and run MongoDB cluster in a named container.
sudo docker run -it --name my-mongo --network my-network -p 27017:27017 mongo

# Download, build, add to network and run Redis server in a named container.
sudo docker run --name my-redis --network my-network -d -p 6379:6379 redis

# Download, build and add to the network a named container with Postgres server.
# Add environment variables (-e) and a volume to persist data.
# You can run it with `sudo docker exec -it my-postgres bash`
sudo docker run --name my-postgres --network my-network -p 5432:5432 -v postgres-data:/var/lib/postgresql/data -e POSTGRES_USER=<username> -e POSTGRES_PASSWORD=<password> -e POSTGRES_DB=<db_name> -d postgres:16

# Download, build and add to the network a named container with Postgres server.
# Add environment variables (-e) and a volume to persist data.
sudo docker run --name mysql-test --network my-network -p 3306:3306 -e MYSQL_ROOT_PASSWORD=<my-password> -d mysql

# Now run it with
sudo docker exec -it mysql-test mysql -u root --password=<my-password>
# Or enter the container with 
sudo docker exec -it mysql-test bash
```
---
#### Sample `Dockerfile` for a python project
```Dockerfile
FROM python: 3.12.3-slim-bookworm

# Prevent storing byte code into *.pyc files on the system.
# Optional, ?POTENTIALLY USELESS?
ENV PYTHONDONTWRITEBYTECODE 1

# Instruct python not to buffer stdout and stderr and deirectly pass it to 
# container stdout and stderr.
ENV PYTHONUNBUFFERED 1

# Create a dir `app` in the root of the container and make it CWD.
WORKDIR /app

COPY requirements.txt /app

RUN pip install --upgrade pip \
		# Prevent pip from creating caches.
		&& pip install --no-cache-dir -r requirements.txt
		# Remove system cache.
		&& rm -rf /root/.cache

# Copy main directory to CWD.
COPY /src /app

# Expose port.
EXPOSE 8000

# Run command
CMD ['python', '-m', ...]
```
---
#### Create an `alias` for a docker image
```bash
# In case you’ve build an image with some default name and don’t want to delete # it an build again you can create an alias with `tag` command.
sudo docker image tag <oldname> <username>/<newname>:<tag>
```
---
#### `Push` local image to `DockerHub`
```bash
# Create an empty repository at your DockerHub account
# Copy the repository name (in form of <username>/<repository_name>)

# On you local machine build the image with your repository name
sudo docker build -t <username>/<repository_name>:<tag>
# OR tag an existing image, creating and alias
sudo docker tag <oldname> <username>/<repository_name>:<tag>

# Login to your docker account on local machine
sudo docker login -u <username>
# ... enter your password

# Push the image
sudo docker push <username>/<reporitory_name>:<tag>
```
----
#### `Remove` unused docker `images`
```bash
# Remove images not bound to containers (usually tagged with `<none>`)
sudo docker image prune
```
---
#### Copy files from container to host and vice-versa
```bash
# Works even if the container is not running
# To host
sudo docker cp <my-container>:/path/to/file.csv /host/path

# To container
sudo docker cp /host/path/file.csv <my-container>:/path/to/source
```
---

