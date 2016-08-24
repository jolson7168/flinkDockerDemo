# Flink Docker Demo
Procedure for building and deploying a Flink Docker container.

## Install Tools
###Java 
(If not already installed...)
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo update-alternatives --config java
echo $JAVA_HOME
java -version
```
###Maven
Needed to build test .jars to be deployed on Flink cluster
```
sudo apt-get install maven
```

###Docker
Needed to build Docker containers
```
sudo apt-get install apt-transport-https ca-certificates
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

# Add Docker repo
sudo vi /etc/apt/sources.list.d/docker.list
 		deb https://apt.dockerproject.org/repo ubuntu-xenial main
sudo apt-get update
sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual

# Install Docker
sudo apt-get install docker-engine
sudo service docker start
sudo docker run hello-world

# Administrative stuff
sudo usermod -aG docker $USER

# Verify!
docker run hello-world	

# Adjust memory and swap accounting
sudo vi /etc/default/grub
	GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
sudo update-grub


# modify DNS
sudo vi /etc/default/docker
	DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"

sudo systemctl enable docker

# Reboot
reboot
```

###Docker - Compose
Needed to instrument the Flink Cluster
```
curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` >/home/ubuntu/docker-compose
(put in path)
chmod +x docker-compose
```

###Flink
Download Flink source in order to build container
```
git clone https://github.com/apache/flink.git
```

###Build Flink Docker container
The configuration file for the build is part of the Flink project
```
cd /flink/flink-contrib/docker-flink
chmod +x build.sh
./build.sh

# Verify build
docker images
# you should see the image built

```


###Bring Flink cluster up locally
Do this just to test, and if you don't have access to a public cloud

```
cd /flink/flink-contrib/docker-flink
docker-compose up -d
```

Verify the Flink cluster is up by going to http://<ip address>:48081/#/overview.
Note that the Docker build file automatically maps the job manager UI port 8081 to 48081

