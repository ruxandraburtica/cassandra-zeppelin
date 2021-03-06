# Cassandra + Spark + Zeppelin

This is a repository for a docker-compose script that creates two Docker containers - one with a Zeppelin instance and the other one with a Cassandra node

## Configuration and Installation
Make sure to have a valid Docker and docker-compose Installation, running on a 64-bit system (either directly on a mac or Linux machine, or on a VirtualBox - or similar - VM running a 64-bit guest; this means that you'll end up running Docker inside a VM, this is fine for testing and learning purposes). 

To install/configure Docker and/or Docker Compose follow the steps described at https://docs.docker.com/compose/install/ and https://docs.docker.com/engine/installation/linux/ubuntu/ (this is for Ubuntu based Linux systems)

## Usage
Once the docker & docker-compose prerequisites are met, go on and clone this repository 
e.g. 
```
git clone https://github.com/academyofdata/cassandra-zeppelin
```
followed by
```
cd cassandra-zeppelin
docker-compose build
docker-compose up -d
```
Assuming that you haven't encountered problems during build or run phase, you can now test that the containers are running by issuing the following command
```
docker ps
```
which should have an output similar with the one below
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                      NAMES
110e8f4b16b3        zeppelin_zeppelin   "bin/zeppelin.sh"        4 days ago          Up 3 days           0.0.0.0:4040->4040/tcp, 0.0.0.0:8080-8081->8080-8081/tcp                                                   zeppelin_zeppelin_1
bbb70c263987        cassandra:3.9       "/docker-entrypoint.s"   4 days ago          Up 3 days           0.0.0.0:7000-7001->7000-7001/tcp, 0.0.0.0:7199->7199/tcp, 0.0.0.0:9042->9042/tcp, 0.0.0.0:9160->9160/tcp   zeppelin_cassandra_1
```
(pay attention in special to the STATUS column - it should say Up and not Exited)
Once the containers are running you can go to http://virtualmachineip:8080 (replace with your own VirtualBox or local machine IP) and you should see the Zeppelin interface

# Bulk-Loading data in Cassandra
To load all the exercise data into a newly created "test" keyspace and creating all the required tables, run the following command inside the Cassandra container (if you have an existing "test" keyspace, drop it)

```
apt-get update && apt-get install -y wget && wget -qO- https://raw.githubusercontent.com/academyofdata/cassandra-zeppelin/master/script.sh | bash
```
(to log into the container run 'docker exec -ti containers_cassandra_1 bash' from your container host, after you check the exact name of your container with 'docker ps -a')

# Starting a Zeppelin only instance

Edit the docker-compose.yml file to read as below
```
zeppelin:
  image:  dylanmei/zeppelin
  environment:
    ZEPPELIN_PORT: 8080
    ZEPPELIN_JAVA_OPTS: >-
      -Dspark.driver.memory=1g
      -Dspark.executor.memory=2g
    SPARK_SUBMIT_OPTIONS: >-
      --conf spark.driver.host=localhost
      --conf spark.driver.port=8081
      
    MASTER: local[*]
  ports:
    - 8080:8080
    - 8081:8081
    - 4040:4040
  volumes:
    - ./znotebooks:/usr/zeppelin/notebook
```
and issue the same docker-compose up -d command
