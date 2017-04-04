# Lab X: Docker Orchestration

Docker Orchestration - In this lab you can play around with the container orchestration features of Docker Engine. You will deploy a Dockerized application to a single host and test the application. You will then configure Docker Swarm Mode and deploy the same application across multiple hosts. You will then see how to scale the application and move the workload across different hosts easily.

> **Difficulty**: Beginner

> **Time**: Approximately 30 minutes

> **Tasks**:
>
> * [Section #1 - Configure Docker with swarm mode](#start-cluster)
> * [Section #2 - Deploy applications across multiple hosts](#multi-application)
> * [Section #3 - Scale the application](#scale-application)
> * [Section #4 - Drain a node and reschedule the containers](#recover-application)

## Document conventions

When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value. 

For instance if you see `ssh <username>@<hostname>` you would actually type something like `ssh ubuntu@node0-a.ivaf2i2atqouppoxund0tvddsa.jx.internal.cloudapp.net`

You will be asked to SSH into various nodes. These nodes are referred to as **node0-a**, **node1-b**, **node2-c**, etc. These tags correspond to the very beginning of the hostnames found on the hands on labs welcome card you were given. 

## <a name="prerequisites"></a>Prerequisites

* Make sure you can connect ssh into the Linux nodes
 
# <a name="start-cluster"></a>Section 1: Configure Swarm Mode

Real-world applications are typically deployed across multiple hosts. This improves application performance and availability, as well as allowing individual application components to scale independently. Docker has powerful native tools to help you do this.

In this section you will configure *Swarm Mode*. This is a new optional mode in which multiple Docker Engines form into a self-orchestrating group of engines called a *swarm*. Swarm mode enables new features such as *services* and *bundles* that help you deploy and manage multi-container apps across multiple Docker hosts.

You will complete the following:

- Configure *Swarm mode*
- Run the app
- Scale the app
- Drain nodes for maintenance and reschedule containers

For the remainder of this lab we will refer to *Docker native clustering* as ***Swarm mode***. The collection of Docker engines configured for Swarm mode will be referred to as the *swarm*.

A swarm comprises one or more *Manager Nodes* and one or more *Worker Nodes*. The manager nodes maintain the state of swarm and schedule appication containers. The worker nodes run the application containers. As of Docker 1.12, no external backend, or 3rd party components, are required for a fully functioning swarm - everything is built-in!

In this part of the demo you will use all three of the nodes in your lab. __node0-a__ will be the Swarm manager, while __node1-b__ and __node2-c__ will be worker nodes. Swarm mode supports a highly available redundant manager nodes, but for the purposes of this lab you will only deploy a single manager node.

## Step 2.1 - Create a Manager node

If you haven't already done so, SSH in to **node0-a**.

```
$ ssh ubuntu@<node0-a IP address>
```

In this step you'll initialize a new Swarm, join a single worker node, and verify the operations worked.

Run `docker swarm init` on **node0-a**.

```
$ docker swarm init
Swarm initialized: current node (rwezvezez3bg1kqg0y0f4ju22) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-1ddw96k6f4efbu1xmcqsk09h8ewr5qse5hmpn6ttngq511yagd-cnhzsicgss51wzv48s72g9oo2 \
    10.0.0.5:2377

Copy the entire `docker swarm join ...` command that is displayed as part of the output from your terminal output. To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Run the `docker info` command to verify that **node0-a** was successfully configured as a swarm manager node.

```
$ docker info
Containers: 2
 Running: 0
 Paused: 0
 Stopped: 2
Images: 2
Server Version: 17.03.1-ee-3
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 13
 Dirperm1 Supported: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: active
 NodeID: rwezvezez3bg1kqg0y0f4ju22
 Is Manager: true
 ClusterID: qccn5eanox0uctyj6xtfvesy2
 Managers: 1
 Nodes: 1
 Orchestration:
  Task History Retention Limit: 5
 Raft:
  Snapshot Interval: 10000
  Number of Old Snapshots to Retain: 0
  Heartbeat Tick: 1
  Election Tick: 3
 Dispatcher:
  Heartbeat Period: 5 seconds
 CA Configuration:
  Expiry Duration: 3 months
 Node Address: 10.0.0.5
 Manager Addresses:
  10.0.0.5:2377
<Snip>
```

The swarm is now initialized with **node0-a** as the only Manager node. In the next section you will add **node1-b** and **node2-c** as *Worker nodes*.

####  Step 2.2 - Join Worker nodes to the Swarm

You will perform the following procedure on **node1-b** and **node2-c**. Towards the end of the procedure you will switch back to **node0-a**.

Open a new SSH session to __node1-b__ (Keep your SSH session to **node0-a** open in another tab or window).

```
$ ssh ubuntu@<node1-b IP address>
```

Now, take that entire `docker swarm join ...` command we copied earlier from `node0-a` where it was displayed as terminal output. We need to paste the copied command into the terminal of **node1-b** and **node2-c**.

It should look something like this for **node1-b**.

```
$ docker swarm join \
    --token SWMTKN-1-1ddw96k6f4efbu1xmcqsk09h8ewr5qse5hmpn6ttngq511yagd-cnhzsicgss51wzv48s72g9oo2 \
    10.0.0.5:2377
```

Again, ssh into **node2-c** and it should look something like this.

```
$ docker swarm join \
    --token SWMTKN-1-1ddw96k6f4efbu1xmcqsk09h8ewr5qse5hmpn6ttngq511yagd-cnhzsicgss51wzv48s72g9oo2 \
    10.0.0.5:2377
```

Once you have run this on **node1-b**, switch back to **node0-a**, and run a `docker node ls` to verify that both nodes are part of the Swarm. You should see three nodes, **node0-a** as the Manager node and **node1-b** and **node2-c** both as Worker nodes.

```
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
5z7q7hqfzndveqnbehkxo2q2r    node1-b   Ready   Active
rwezvezez3bg1kqg0y0f4ju22 *  node0-a   Ready   Active        Leader
u4wc3lcivogmms3cjtr1neh40    node2-c   Ready   Active
```

The `docker node ls` command shows you all of the nodes that are in the swarm as well as their roles in the swarm. The `*` identifies the node that you are issuing the command from.

Congratulations. You have configured a swarm with one manager node and two worker nodes.

# <a name="multi-application"></a>Section 2: Deploy applications across multiple hosts

Now that you have a swarm up and running, it is time to deploy an app to it. To do this you will complete the following actions:

- Create a new overlay network for the application
- Deploy the application components as Docker *services*

You will perform the following procedure from **node0-a**.

### Step 3.1 - Create a new overlay network for the application

1. Create a new overlay network called `catnet`. `catnet` will be the overlay network that our application containers live on.

   ```
   labuser@node0-a:~/$ docker network create -d overlay catnet
   81pjggkd57fbbl7zmheasc6m8
   ```

   The network ID will be output to signal the succesful creation of a Docker network. The `-d` flag let's you specify which network driver to use. In this example you are telling Docker to create a new network using the **overlay** driver. In *Swarm Mode* the overlay network does not require an external key-value store. It is integrated into the engine. The overlay provides reachability between the hosts across the underlay network. Out of the box containers on the same overlay network will be able to ping eachother without any other special configuration.

2. Confirm that the network was created. We can see that there are several other default networks that already exist in a Docker host.

   ```
   labuser@node0-a:~/$ docker network ls
	NETWORK ID          NAME                DRIVER              SCOPE
	09b78531f1d5        bridge              bridge              local
	e7cf5b545252        docker_gwbridge     bridge              local
	9311e4b9e4e1        host                host                local
	bhpf10k7w3gt        ingress             overlay             swarm
	26b729bf0d65        none                null                local

	```

Now that the container network is created you are ready to deploy the application to the swarm.

#### Step 3.2 - Deploy the application components as Docker services

Your `cats` application is becoming very popular on the internet. People just love pictures of cats. You are going to have to scale your application to meet peak demand. You will have to do this across multiple hosts for high availability. We will use the concept of *Services* to scale our application easily and manage many containers as a single entity.

*Services* are a new concept in Docker 1.12. They work with swarms and are intended for long-running containers.


You will perform this procedure from **node0-a**.

1. Deploy `cats` as a *Service*. Remember to use your `<Docker ID>` instead of `markchurch`. However, if you did not complete Part 1 of this lab you are welcome to use `markchurch/cats` as the application image as that will work as well.

   ```bash
   labuser@node0-a:~/$ docker service create --name cat-app --network catnet -p 8000:5000 markchurch/cats
5qwkfpdpsm72opbxvzgzk9s5q
   ```


2. Verify that the `service create` has been received by the Swarm manager.

   ```
   labuser@node0-a:~/$ docker service ls
	ID            NAME     SCALE  IMAGE            COMMAND
	5qwkfpdpsm72  cat-app  1      markchurch/cats
   ```

3. We can insepct the progress of our individual service containers by using the `docker service tasks` command. It may take a minute or two until the `LAST STATE` column is in the `Running` status.

	```
	labuser@node0-a:~/$ docker service tasks cat-app
	ID                         NAME       SERVICE  IMAGE            LAST STATE         DESIRED STATE  NODE
	ehfg9vq9m8bgyrxyv0abrex8c  cat-app.1  cat-app  markchurch/cats  Running 5 minutes  Running        node0-a-9128f1906df54acda5044f56a1a86b07-2
	```
	
	The state of the service may change a couple times until it is running. The image is being downloaded from your Docker Hub repository directly to the other engines in the Swarm. Once the image is downloaded the container goes into a running state on one of the three nodes.

	At this point it may not seem that we have done anything very differently than in part 1. We have again deployed a single container on a host and mapped it to port `8000`. The difference here is that the container has been scheduled on a swarm cluster.  The application is available on port `8000` on any of the hosts in the cluster even though the `cat-app` container is just running on a single host. The Swarm intelligently routes routes requests to the `cat-app`.

4. Check that the app is running in your browser. You can use any of the node URLs. The Swarm will advertise the published port on every host in the cluster.

   Point your web browser to `http://<node0-a-public-ip>:8000`. Now try pointing it to `http://<node1-b-public-ip>:8000`. You should see the same `cat-app` being served by the same container ID in both cases. Your request is being routed by the Docker engine from any Swarm node to the correct node. (The picture will change but the container ID confirms to us that the same container is being hit every time.)

Well done. You have deployed the cat-app to your new Swarm using Docker services. 

#### Step 3.3 - Scale the app

Demand is crazy! Everybody loves your `cats` app! It's time to scale out.

One of the great things about *services* is that you can scale them up and down to meet demand. In this step you'll scale the web front-end service up and then back down.

You will perform the following procedure from **node0-a**.

1. Scale the number of containers in the **web** service to 7 with the `docker service update --replicas` command. `replicas` is the term we use to describe identical containers providing the same service.

   ```bash
   labuser@node0-a:~/$ docker service update --replicas 7 cat-app
   cat-app
   ```
	The Swarm manager schedules so that there are 7 `cat-app` containers in the cluster. These will be scheduled evenly across the Swarm members.

2. We are going to use the `docker service tasks` command again but pair it with the `watch` command so we can see the containers come up in real time

   ```bash
   labuser@node0-a:~/$ watch docker service tasks cat-app
   Every 2.0s: docker service tasks cat-app                                                                                                                                            Sat Jun 18 21:28:01 2016

	ID                         NAME       SERVICE  IMAGE            LAST STATE          DESIRED STATE  NODE
	ehfg9vq9m8bgyrxyv0abrex8c  cat-app.1  cat-app  markchurch/cats  Running 17 minutes  Running        node0-a-9128f1906df54acda5044f56a1a86b07-2
	868i4miwqutkla8oih3tvrifi  cat-app.2  cat-app  markchurch/cats  Running 2 minutes   Running        node1-b-9128f1906df54acda5044f56a1a86b07-2
	1dahwhg1ma3oy2wgchlfof4ad  cat-app.3  cat-app  markchurch/cats  Running 2 minutes   Running        node2-c-9128f1906df54acda5044f56a1a86b07-2
	3hiymz9dbna33gyzevmrcqq6s  cat-app.4  cat-app  markchurch/cats  Running 2 minutes   Running        node2-c-9128f1906df54acda5044f56a1a86b07-2
	01s73ggiaub4om2d4u4wkl1cb  cat-app.5  cat-app  markchurch/cats  Running 2 minutes   Running        node0-a-9128f1906df54acda5044f56a1a86b07-2
	4bx4v9t7c9ujifusj3vd6nq10  cat-app.6  cat-app  markchurch/cats  Running About a minute  Running        node1-b-9128f1906df54acda5044f56a1a86b07-2
	7s57r3capvz77rp5ejhalbov5  cat-app.7  cat-app  markchurch/cats  Running About a minute  Running        node2-c-9128f1906df54acda5044f56a1a86b07-2   

	```

   Notice that there are now 7 containers listed. It may take a few seconds for the new containers in the service to all show as **RUNNING**.  The `NODE` column tells us on which node a container is running. After all the containers are in `RUNNING` state press `CTRL-C` and it will take you out of the `watch` screen.

3. Confirm the scaling operation and container load-balancing from a web browser.

   Point your web browser to `http://<any-node-ip>:8000` and hit the refresh button a few times. You will see the container ID changing. This is because the swarm is automatically load balancing across all seven containers. You can send a request to any Swarm member and you will hit any container in the service.

4. Scale the service back down just five containers again with the `docker service update --replicas` command. 

   ```bash
   labuser@node0-a:~/$ docker service update --replicas 5 cat-app
   web
   ```

5. Verify that the number of containers has been reduced to 5 using the `docker service tasks cat-app` command and your web browser.

You have successfully scaled a swarm service up and down.

#### Step 3.4 - Bring a node down for maintenance.

Your cat-app has been doing amazing. It's number 1 on the Apple Store! You have scaled up during the holidays and down during the slow season. Now you are doing maintenance on one of your servers so you will need to gracefully take a server out of the swarm without interrupting service to your customers.


1. Take a look at the status of your nodes again so you can get the node names for step 2.  

   ```bash
   labuser@node0-a:~/$ docker node ls
	ID                           NAME                                          MEMBERSHIP  STATUS  AVAILABILITY  MANAGER 	STATUS  LEADER
	5zkcnyq6uqxv4sfpzsyvs8x3s    node1-b-9128f1906df54acda5044f56a1a86b07-2  Accepted    Ready   Active
	acu262y2ight8uyn6fg8yvust *  node0-a-9128f1906df54acda5044f56a1a86b07-2  Accepted    Ready   Active        Reachable       Yes
	brr8ri58aa1ed9051drszoyh0    node2-c-9128f1906df54acda5044f56a1a86b07-2  Accepted    Ready   Active
   ```

   You will be taking node 2 out of service for maintenance

2. SSH in to node 2 to see the containers that you have running there.

   ```bash
   labuser@node2-c:~/$ docker ps
	CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS               	NAMES
	c90e11f21515        markchurch/cats:latest   "python ./app.py"   4 minutes ago       Up 4 minutes        5000/tcp            cat-app.7.7s57r3capvz77rp5ejhalbov5
	475bcb6ced7c        markchurch/cats:latest   "python ./app.py"   13 minutes ago      Up 13 minutes       5000/tcp            cat-app.3.1dahwhg1ma3oy2wgchlfof4ad
   ```

   You can see that we have two of the cat-app containers running here.

3. Now let's SSH back in to node0 (the Swarm manager) and take node2 out of service. You will insert your node ID of node 2 that you got in step 1.

   ```
   labuser@node0-a:~/$ docker node update --availability drain node2-c-9128f1906df54acda5044f56a1a86b07-2
	node2-c-9128f1906df54acda5044f56a1a86b07-2

   ```
4. Check the status of the nodes

	```bash
	labuser@node0-a:~/$ docker node ls
	ID                           NAME                                          MEMBERSHIP  STATUS  AVAILABILITY  MANAGER 	STATUS  LEADER
	5zkcnyq6uqxv4sfpzsyvs8x3s    node1-b-9128f1906df54acda5044f56a1a86b07-2  Accepted    Ready   Active
	acu262y2ight8uyn6fg8yvust *  node0-a-9128f1906df54acda5044f56a1a86b07-2  Accepted    Ready   Active        Reachable       Yes
	brr8ri58aa1ed9051drszoyh0    node2-c-9128f1906df54acda5044f56a1a86b07-2  Accepted    Ready   Drain
	```
	Node 2 is now in the `Drain` state. 


5. Switch back to node2 and see what is running there.

	```bash
	labuser@node2-c:~/$ docker ps
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
	labuser@node2-c:~/$
	```

   **node-2** does not have any containers running on it.

6. Lastly, check the service again on node 0 to make sure that the container were rescheduled. You should see all five containers running on the remaining two nodes.

	```bash
   labuser@node0-a:~/$ docker service tasks cat-app
ID                         NAME       SERVICE  IMAGE            LAST STATE          DESIRED STATE  NODE
2g4z7c7ykbdgbrbpt3kgekdrc  cat-app.2  cat-app  markchurch/cats  Running 10 minutes  Running        node0-a-9128f1906df54acda5044f56a1a86b07-2
79cil3duim38z68troik84bwn  cat-app.3  cat-app  markchurch/cats  Running 4 minutes   Running        node1-b-9128f1906df54acda5044f56a1a86b07-2
30t4w3do776zfybq1ffwkx3g6  cat-app.4  cat-app  markchurch/cats  Running 10 minutes  Running        node0-a-9128f1906df54acda5044f56a1a86b07-2
01s73ggiaub4om2d4u4wkl1cb  cat-app.5  cat-app  markchurch/cats  Running 19 minutes  Running        node0-a-9128f1906df54acda5044f56a1a86b07-2
4bx4v9t7c9ujifusj3vd6nq10  cat-app.6  cat-app  markchurch/cats  Running 10 minutes  Running        node1-b-9128f1906df54acda5044f56a1a86b07-2
```

7. Lastly, go back to your browser and verify after that everything is running fine after our maintenance operation.

Congratulations! You've completed this lab. You now know how to build a swarm, deploy applications as collections of services, and scale individual services up and down.