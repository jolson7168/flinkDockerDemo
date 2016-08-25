# Flink Docker Demo
Procedure for building and deploying a Flink Docker container.

Notes are designed to be run on Ubuntu 16.04, and were executed on that build of Ubuntu virtualized under Parallels (Mac OS).

## Install Tools
###open ssh
```
sudo apt-get install openssh-server
```

###Install git
```
sudo apt-get install git
```

###update PATH in .bashrc
```
export PATH=$PATH:/home/ubuntu
```

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

#Reset session
Logout, and back in again

# Verify any user can run!
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
# Remember to put in PATH!
curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` >/home/ubuntu/docker-compose
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
cd ~/flink/flink-contrib/docker-flink
chmod +x build.sh
./build.sh

# Verify build
docker images

```

###Bring Flink cluster up locally
Do this just to test, and if you don't have access to a public cloud

```
cd ~/flink/flink-contrib/docker-flink
docker-compose up -d
```

Verify the Flink cluster is up by going to http://10.211.55.53:48081/#/overview.
Note that the Docker build file automatically maps the job manager UI port 8081 to 48081

###Build WordCount .jar to deploy to test cluster
```
cd ~/flink/flink-examples/flink-examples-batch
mvn clean package
```

Put some test data on the Flink cluster
```
docker exec -it dockerflink_taskmanager_1 bash
wget -O /tmp/hamlet.txt http://www.gutenberg.org/cache/epub/1787/pg1787.txt
```

###Run WordCount job on test data via GUI
Go to http://10.211.55.53:48081/#/overview
- Upload jar 
	cd ~/flink/flink-examples/flink-examples-batch
- Job arguments
	--input file:///tmp/hamlet.txt --output file:///tmp/out.txt

###Verify job has run
Get on taskmanager1
```
docker exec -it dockerflink_taskmanager_1 bash
more /tmp/output.txt
```

###Scale the cluster!
Add another node

```
cd ~/flink/flink-contrib/docker-flink
docker-compose scale taskmanager=3
```

###Kill the cluster
docker-compose kill


### Install Cloud Foundry CLI Tools:
Needed to interact with IBM's Bluemix

```
# Remember to put in PATH!
curl -L "https://cli.run.pivotal.io/stable?release=linux64-binary&source=github" | tar -zx
```

###Install the Cloud Foundry IBM Containers plug-in
Needed to interact with Bluemix's container store
```
cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-linux_x64
```

###Login to Bluemix
```
cf login -a https://api.ng.bluemix.net
<login> / <pw>
```

###Update the Docker - Compose .yml file to point to Bluemix.

```
cd ~/flink/flink-contrib/docker-flink
mv docker-compose.yml docker-compose-old.yml
vi docker-compose.yml

version: "2"
services:
  jobmanager:
    image: registry.ng.bluemix.net/chiflink2/flink
    ports:
      - "8081:8081"
      - "22:22"
      - "8080:8080"
      - "6123:6123"
    command: jobmanager
    volumes:
      - /opt/flink/conf

  taskmanager:
    image: registry.ng.bluemix.net/chiflink2/flink
    ports:
      - "22:22"
      - "6122:6122"
      - "6123:6123"
    depends_on:
      - jobmanager
    command: taskmanager
    volumes_from:
      - jobmanager:ro
```

###Log in to BM container

```
cf ic login
cf ic namespace set chiflink2 # If needed....
```

Update the Docker environment variables to point at BM's container store

List docker containers on BM
```
cf ic images
```

###Upload Flink container to BM

```
docker tag flink registry.ng.bluemix.net/chiflink2/flink
docker push registry.ng.bluemix.net/chiflink2/flink
```

Verify the containers is now on BM
```
cf ic images
```

###Bring up the Flink cluster
```
docker-compose up -d --force-recreate
```

###Request and bind public IP address to jobmanager
```
cf ic ip request
cf ic ip list
cf ic ip bind <ip> dockerflink_jobmanager_1
```

###Verify all ports / IP addresses are legit on BM console
Login to https://console.ng.bluemix.net

###Temp Fix! Latest build corrupt?
Fix on the taskmanagers
```
docker exec -it <id> /bin/sh
cd /opt/flink-1.1.1/conf/fink-conf.yaml
replace localhost with ip address of jobmaster
```	

###Test public URL on Flink job manager
URL: http:/<ip>:8081

###Run another job
Similar as above


### Finally, bring down cluster
```
docker-compose kill
```