# swarm-jobs-multipass
Docker Swarm Job support is almost here. Let's test it out.

Support for running "one-off" jobs in Docker Swarm is one of the most long awaited features in Swarm. The code will be included in Docker 20.03.0 but is available now for testing.  Using [Multipass](https://multipass.run) you can setup a 3 node Docker Swarm in a few minutes and start playing about with Swarm Jobs.

Disclaimer: These notes are kinda rough and it is late, so apologies.  I'll clean it up later.

## Install Multipass

Instructions can be found [here](https://multipass.run).

## Download a Recent Docker Snapshot

Snapshots are available [here](https://github.com/AkihiroSuda/moby-snapshot/releases)

Since we are planning to run our Swarm cluster on Ubuntu VMs let's download the .deb [packages](https://github.com/AkihiroSuda/moby-snapshot/releases/download/snapshot-20200818/moby-snapshot-ubuntu-focal-x86_64-deb.tbz) and extract the .tbz bundle using : tar -xvf moby-snapshot-ubuntu-focal-x86_64-deb.tbz

## Start 3 VMs

```shell
multipass launch --mem 1G --cpus 1 --disk 10G --name swarm-node-1 20.04
multipass launch --mem 1G --cpus 1 --disk 10G --name swarm-node-2 20.04
multipass launch --mem 1G --cpus 1 --disk 10G --name swarm-node-3 20.04

Mount in the HOME directory from the host to the VM :

multipass mount $HOME swarm-node-1
multipass mount $HOME swarm-node-2
multipass mount $HOME swarm-node-3

On MacOS the above command will mount /Users/<username> to /Users/<username> inside each VM.

```

## Install and Start Docker

Login to each VM, install the Docker .deb packages and start docker.

```shell
multipass shell swarm-node-1

cd /Users/<username>
sudo apt-get install ./*.deb
sudo dockerd
```

Repeat the above steps on swarm-node-2 and swarm-node-3.

At this point we can leave the terminals as they are. Docker is running on each.

## Setup the Swarm Cluster

Start 3 new terminals and multipass shell into each VM.

swarm-node-1 :
```
sudo docker swarm init
```
You will get a string like "docker swarm join --token". Copy that string which has a unique token in it and run this command on swarm-mode-2 and swarm-mode-3.

At this point we have a 3 node cluster setup. To confirm this run the following on swarm-node-1 :
```
docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
l07dxnibdqm01ln861b38toko *   swarm-node-1        Ready               Active              Leader              0.0.0-20200817150913-f3c88f0
jkjr1vzw4vyjfy5w7626xpy3j     swarm-node-2        Ready               Active                                  0.0.0-20200817150913-f3c88f0
ne5xm55tefl3c0nfil3xcgv65     swarm-node-3        Ready               Active                                  0.0.0-20200817150913-f3c88f0
```

Excellent. Let's try running a Swarm Job.

Details on what Jobs are and how to run them can be found [here](https://github.com/dperny/cli/blob/9375644e34f6c991244323d708048ed2efb8d3ad/docs/reference/commandline/service_create.md#running-as-a-job).

### Running a Global Job

This command will run a "one-off" globally which means each node in the cluster will run a task for this job.
```
docker service create --name ping-google-global --mode=global-job bash ping -c 5 google.com

j0pkvsbm2wqsfvp3jgyf38yt4
job progress: 0 out of 0 complete 
job complete

View status of Global Job :

docker service ps ping-google-global
ID                  NAME                                           IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
cnoa4e8qm5vr        ping-google-global.jkjr1vzw4vyjfy5w7626xpy3j   bash:latest         swarm-node-2        Complete            Complete 7 seconds ago                       
o00xnaizqp1c        ping-google-global.l07dxnibdqm01ln861b38toko   bash:latest         swarm-node-1        Complete            Complete 7 seconds ago                       
u7onbs7mf10b        ping-google-global.ne5xm55tefl3c0nfil3xcgv65   bash:latest         swarm-node-3        Complete            Complete 7 seconds ago               

View Logs :

docker service logs ping-google-global

ping-google-global.0.u7onbs7mf10b@swarm-node-3    | PING google.com (74.125.193.101): 56 data bytes
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | 64 bytes from 74.125.193.101: seq=0 ttl=105 time=10.990 ms
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | 64 bytes from 74.125.193.101: seq=1 ttl=105 time=18.566 ms
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | 64 bytes from 74.125.193.101: seq=2 ttl=105 time=12.483 ms
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | 64 bytes from 74.125.193.101: seq=3 ttl=105 time=8.755 ms
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | 64 bytes from 74.125.193.101: seq=4 ttl=105 time=10.792 ms
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | 
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | --- google.com ping statistics ---
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | 5 packets transmitted, 5 packets received, 0% packet loss
ping-google-global.0.u7onbs7mf10b@swarm-node-3    | round-trip min/avg/max = 8.755/12.317/18.566 ms
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | PING google.com (74.125.193.101): 56 data bytes
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | 64 bytes from 74.125.193.101: seq=0 ttl=105 time=12.188 ms
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | 64 bytes from 74.125.193.101: seq=1 ttl=105 time=12.629 ms
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | 64 bytes from 74.125.193.101: seq=2 ttl=105 time=17.234 ms
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | 64 bytes from 74.125.193.101: seq=3 ttl=105 time=16.733 ms
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | 64 bytes from 74.125.193.101: seq=4 ttl=105 time=12.029 ms
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | 
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | --- google.com ping statistics ---
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | 5 packets transmitted, 5 packets received, 0% packet loss
ping-google-global.0.cnoa4e8qm5vr@swarm-node-2    | round-trip min/avg/max = 12.029/14.162/17.234 ms
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | PING google.com (74.125.193.101): 56 data bytes
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | 64 bytes from 74.125.193.101: seq=0 ttl=105 time=10.550 ms
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | 64 bytes from 74.125.193.101: seq=1 ttl=105 time=12.339 ms
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | 64 bytes from 74.125.193.101: seq=2 ttl=105 time=9.808 ms
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | 64 bytes from 74.125.193.101: seq=3 ttl=105 time=11.686 ms
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | 64 bytes from 74.125.193.101: seq=4 ttl=105 time=21.981 ms
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | 
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | --- google.com ping statistics ---
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | 5 packets transmitted, 5 packets received, 0% packet loss
ping-google-global.0.o00xnaizqp1c@swarm-node-1    | round-trip min/avg/max = 9.808/13.272/21.981 ms

```

### Running a Replicated Job

The next command will run a Replicated Job across the cluster.  10 tasks will run across the 3 nodes and a maximum of 2 tasks will run at a time.

```
docker service create --name ping-google --replicas=10 --max-concurrent=2  --mode=replicated-job bash ping -c 2 google.com
rviqj9z5qh9hufntn01i9kz95
job progress: 10 out of 10 complete [==================================================>] 
active tasks: 0 out of 0 tasks 
1/10: complete  [==================================================>] 
2/10: complete  [==================================================>] 
3/10: complete  [==================================================>] 
4/10: complete  [==================================================>] 
5/10: complete  [==================================================>] 
6/10: complete  [==================================================>] 
7/10: complete  [==================================================>] 
8/10: complete  [==================================================>] 
9/10: complete  [==================================================>] 
10/10: complete  [==================================================>] 
job complete

docker service ps ping-google

ID                  NAME                                    IMAGE               NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
it3a3jcr4b4l        ping-google.1                           bash:latest         swarm-node-2        Complete            Complete 17 seconds ago                       
m6z8ozurvbjn        ping-google.2                           bash:latest         swarm-node-2        Complete            Complete 15 seconds ago                       
xgkzabektg7d        ping-google.3                           bash:latest         swarm-node-1        Complete            Complete 15 seconds ago                       
mz6gnbgz7by3        ping-google.4                           bash:latest         swarm-node-2        Complete            Complete 13 seconds ago                       
uatdczq994b5        ping-google.5                           bash:latest         swarm-node-1        Complete            Complete 13 seconds ago                       
esxlxtvzpzcf        ping-google.6                           bash:latest         swarm-node-2        Complete            Complete 11 seconds ago                       
nbzop0dmbjwp        ping-google.7                           bash:latest         swarm-node-1        Complete            Complete 11 seconds ago                       
ssdv982lqfqd        ping-google.8                           bash:latest         swarm-node-2        Complete            Complete 9 seconds ago                        
qqilntkr16rm        ping-google.9                           bash:latest         swarm-node-1        Complete            Complete 9 seconds ago                        
xcuinnwou2fb        ping-google.l07dxnibdqm01ln861b38toko   bash:latest         swarm-node-1        Complete            Complete 17 seconds ago    
```
