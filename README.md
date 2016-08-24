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

