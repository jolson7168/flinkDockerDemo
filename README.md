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
(remember to put in path!)
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

###Build WordCount .jar to deploy to test cluster
```
cd ~/flink/flink-examples/flink-examples-batch
mvn clean package
```

Put some test data on the Flink cluster
```
docker exec -it dockerflink_jobmanager_1 bash
wget -O /tmp/hamlet.txt http://www.gutenberg.org/cache/epub/1787/pg1787.txt
```

###Run WordCount job on test data via GUI
Go to http://<ip address>:48081/#/overview
- Upload jar 
	cd ~/flink/flink-examples/flink-examples-batch
- Job arguments
	input file:///tmp/hamlet.txt --output file:///tmp/out.txt

###Verify job has run
Get on taskmanager1
```
docker exec -it dockerflink_taskmanager_1 bash
more /tmp/output.txt
```

###Kill the cluster
docker-compose kill


### Install Cloud Foundry CLI Tools:
Needed to interact with IBM's Bluemix

```
curl -L "https://cli.run.pivotal.io/stable?release=linux64-binary&source=github" | tar -zx
(remember to put in path!)
```

###Install the Cloud Foundry IBM Containers plug-in
Needed to interact with Bluemix's container store
```
cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-linux_x64
```

###Login to Bluemix
```
cf login -a https://api.ng.bluemix.net
technology@nododos.com / LoganCalifornia2016
```

###Update the Docker - Compose .yml file to point to Bluemix.

```
cd ~/flink/flink-contrib/docker-flink
mv docker-compose.yml docker-compose-old.yml
vi docker-compose.yml

	docker-compose-bluemix.yml

	version: "2"
	services:
	  jobmanager:
	    image: registry.ng.bluemix.net/nododos/flink
	    ports:
	      - "48081:8081"
	    command: jobmanager
	    volumes:
	      - /opt/flink/conf

	  taskmanager:
	    image: registry.ng.bluemix.net/nododos/flink
	    depends_on:
	      - jobmanager
	    command: taskmanager
	    volumes_from:
	      - jobmanager:ro
```

###Log in to BM container

```
cf ic login
```

Update the Docker environment variables to point at BM's container store

List docker containers on BM
```
cf ic images
```

###Upload Flink container to BM

```
docker tag flink registry.ng.bluemix.net/nododos/flink
docker push registry.ng.bluemix.net/nododos/flink
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

ssh into jobmanager
	docker exec -it $(docker ps --filter name=dockerflink_taskmanager_3 --format={{.ID}}) /bin/sh

fix config
	ssh into machine 
	cd /opt/flink-1.1.1/conf/fink-conf.yaml
	replace localhost with ip address of jobmaster
	




Scale Flink
	docker-compose scale taskmanager=3

Bring down Flink
	docker-compose -f docker-compose-bluemix.yml kill