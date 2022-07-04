# Jenkins

## Installation Ubuntu

Long Term Support release A LTS (Long-Term Support) release is chosen every 12 weeks from the stream of regular releases as the stable release for that time period. It can be installed from the debian-stable apt repository.
```
Step 1: Install Java
$ sudo apt update
$ sudo apt install openjdk-8-jdk
Step 2: Add Jenkins Repository
$ wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key
| sudo apt-key add –
Step 3: Add Jenkins repo to the system
$ sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable
binary/ > /etc/apt/sources.list.d/jenkins.list’
Step 4: Install Jenkins
$ sudo apt update
$ sudo apt install Jenkins
Step 5: Verify installation
$ systemctl status Jenkins
Step 6: Once Jenkins is up and running, access it from the
link:
http://localhost:8080
```

### The package installation will:

- Setup Jenkins as a daemon launched on start. Run systemctl cat jenkins for more details.

- Create a ‘jenkins’ user to run this service.

- Direct console log output to systemd-journald. Run journalctl -u jenkins.service if you are troubleshooting Jenkins.

- Populate /lib/systemd/system/jenkins.service with configuration parameters for the launch, e.g JENKINS_HOME

- Set Jenkins to listen on port 8080. Access this port with your browser to start configuration.

**Note:** If Jenkins fails to start because a port is in use, run systemctl edit jenkins and add the following:
```
    [Service]
      Environment="JENKINS_PORT=8081"
```
Here, "8081" was chosen but you can put another port available.
## Installation Docker
```
docker network create jenkins

docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```
1. ( Optional ) Specifies the Docker container name to use for running the image. By default, Docker will generate a unique name for the container.
2. ( Optional ) Automatically removes the Docker container (the instance of the Docker image) when it is shut down.
3. ( Optional ) Runs the Docker container in the background. This instance can be stopped later by running docker stop jenkins-docker.
4. Running Docker in Docker currently requires privileged access to function properly. This requirement may be relaxed with newer Linux kernel versions.
5. This corresponds with the network created in the earlier step.
6. Makes the Docker in Docker container available as the hostname docker within the jenkins network.
7. Enables the use of TLS in the Docker server. Due to the use of a privileged container, this is recommended, though it requires the use of the shared volume described below. This environment variable controls the root directory where Docker TLS certificates are managed.
8. Maps the /certs/client directory inside the container to a Docker volume named jenkins-docker-certs as created above.
9. Maps the /var/jenkins_home directory inside the container to the Docker volume named jenkins-data. This will allow for other Docker containers controlled by this Docker container’s Docker daemon to mount data from Jenkins.
( Optional ) Exposes the Docker daemon port on the host machine. This is useful for executing docker commands on the host machine to control this inner Docker daemon.
10. The docker:dind image itself. This image can be downloaded before running by using the command: docker image pull docker:dind.
11. The storage driver for the Docker volume. See "Docker storage drivers" for supported options.

```
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.346.1-1
  ```

Simple command
```
 docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
 ```
 Docker in Jenkins
 ```
 docker run -p 8080:8080 -p 50000:50000 -d \
 -v jenkins_home:/var/jenkins_home \
 -v /var/run/docker.sock:/var/run/docker.sock \
 -v $(which docker):/usr/bin/docker \
 jenkins/jenkins:lts
 ```
Change the permission to allow RW access from the Jenkins container
```
chmod 666 /var/run/docker.sock
