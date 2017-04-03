# Lab X: Docker Networking

Quick description of the lab.

> **Difficulty**: Beginner to Intermediate

> **Time**: Approximately 60 minutes

> **Tasks**:
>
> * [Prerequisites](#prerequisites)
> * [Networking Basics](#task1)
> * [Bridge Networking](#task2)
> * [Overlay Networking](#task3)
> * [HTTP Routing Mesh](#task4)

## Document conventions

When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value. 

For instance if you see `ssh <username>@<hostname>` you would actually type something like `ssh labuser@v111node0-adaflds023asdf-23423kjl.appnet.com`

You will be asked to SSH into various nodes. These nodes are referred to as **v111node0**, **v111node1** etc. These tags correspond to the very beginning of the hostnames you will find in your welcome email. 

## <a name="prerequisites"></a>Prerequisites

- Lists all your prerequisites

## <a name="Task 1"></a>Task #1

The `docker network` command is the main command for configuring and managing container networks.

Run a simple `docker network` command from any of your lab machines.

```
$ docker network

Usage:  docker network COMMAND

Manage Docker networks

Options:
      --help   Print usage

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```

The command output shows how to use the command as well as all of the `docker network` sub-commands. As you can see from the output, the `docker network` command allows you to create new networks, list existing networks, inspect networks, and remove networks. It also allows you to connect and disconnect containers from networks.

# <a name="list_networks"></a>Step 2: List networks

Run a `docker network ls` command to view existing container networks on the current Docker host.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1befe23acd58        bridge              bridge              local
726ead8f4e6b        host                host                local
ef4896538cc7        none                null                local
```

The output above shows the container networks that are created as part of a standard installation of Docker.

New networks that you create will also show up in the output of the `docker network ls` command.

You can see that each network gets a unique `ID` and `NAME`. Each network is also associated with a single driver. Notice that the "bridge" network and the "host" network have the same name as their respective drivers.

# <a name="inspect"></a>Step 3: Inspect a network

The `docker network inspect` command is used to view network configuration details. These details include; name, ID, driver, IPAM driver, subnet info, connected containers, and more.

Use `docker network inspect` to view configuration details of the container networks on your Docker host. The command below shows the details of the network called `bridge`.

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "1befe23acd58cbda7290c45f6d1f5c37a3b43de645d48de6c1ffebd985c8af4b",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

> **NOTE:** The syntax of the `docker network inspect` command is `docker network inspect <network>`, where `<network>` can be either network name or network ID. In the example above we are showing the configuration details for the network called "bridge". Do not confuse this with the "bridge" driver.


# <a name="list_drivers"></a>Step 4: List network driver plugins

The `docker info` command shows a lot of interesting information about a Docker installation.

Run a `docker info` command on any of your Docker hosts and locate the list of network plugins.

```
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.12.3
Storage Driver: aufs
<Snip>
Plugins:
 Volume: local
 Network: bridge host null overlay    <<<<<<<<
Swarm: inactive
Runtimes: runc
<Snip>
```

The output above shows the **bridge**, **host**, **null**, and **overlay** drivers.

## <a name="Task #2"></a>Task #2

Every clean installation of Docker comes with a pre-built network called **bridge**. Verify this with the `docker network ls` command.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1befe23acd58        bridge              bridge              local
726ead8f4e6b        host                host                local
ef4896538cc7        none                null                local
```

The output above shows that the **bridge** network is associated with the *bridge* driver. It's important to note that the network and the driver are connected, but they are not the same. In this example the network and the driver have the same name - but they are not the same thing!

The output above also shows that the **bridge** network is scoped locally. This means that the network only exists on this Docker host. This is true of all networks using the *bridge* driver - the *bridge* driver provides single-host networking.

All networks created with the *bridge* driver are based on a Linux bridge (a.k.a. a virtual switch).

Install the `brctl` command and use it to list the Linux bridges on your Docker host.

```
# Install the brctl tools

$ apt-get install bridge-utils
<Snip>

# List the bridges on your Docker host

$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242f17f89a6       no
```  

The output above shows a single Linux bridge called **docker0**. This is the bridge that was automatically created for the **bridge** network. You can see that it has no interfaces currently connected to it.

You can also use the `ip` command to view details of the **docker0** bridge.

```
$ ip a
<Snip>
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:f1:7f:89:a6 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:f1ff:fe7f:89a6/64 scope link
       valid_lft forever preferred_lft forever
```

# <a name="connect-container"></a>Step 2: Connect a container

The **bridge** network is the default network for new containers. This means that unless you specify a different network, all new containers will be connected to the **bridge** network.

Create a new container.

```
$ docker run -dt ubuntu sleep infinity
6dd93d6cdc806df6c7812b6202f6096e43d9a013e56e5e638ee4bfb4ae8779ce
```

This command will create a new container based on the `ubuntu:latest` image and will run the `sleep` command to keep the container running in the background. As no network was specified on the `docker run` command, the container will be added to the **bridge** network.

Run the `brctl show` command again.

```
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242f17f89a6       no              veth3a080f
```

Notice how the **docker0** bridge now has an interface connected. This interface connects the **docker0** bridge to the new container just created.

Inspect the **bridge** network again to see the new container attached to it.

```
$ docker network inspect bridge
<Snip>
        "Containers": {
            "6dd93d6cdc806df6c7812b6202f6096e43d9a013e56e5e638ee4bfb4ae8779ce": {
                "Name": "reverent_dubinsky",
                "EndpointID": "dda76da5577960b30492fdf1526c7dd7924725e5d654bed57b44e1a6e85e956c",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
<Snip>
```

# <a name="ping_local"></a>Step 3: Test network connectivity

The output to the previous `docker network inspect` command shows the IP address of the new container. In the previous example it is "172.17.0.2" but yours might be different.

Ping the IP address of the container from the shell prompt of your Docker host. Remember to use the IP of the container in **your** environment.

```
$ ping 172.17.0.2
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.069 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.052 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.050 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.049 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.049 ms
^C
--- 172.17.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 3999ms
rtt min/avg/max/mdev = 0.049/0.053/0.069/0.012 ms
```

Press `Ctrl-C` to stop the ping. The replies above show that the Docker host can ping the container over the **bridge** network.

Log in to the container, install the `ping`
 program and ping `www.docker.com`.

 ```
# Get the ID of the container started in the previous step.
$ docker ps
CONTAINER ID    IMAGE    COMMAND             CREATED  STATUS  NAMES
6dd93d6cdc80    ubuntu   "sleep infinity"    5 mins   Up      reverent_dubinsky

# Exec into the container
$ docker exec -it 6dd93d6cdc80 /bin/bash

# Update APT package lists and install the iputils-ping package
root@6dd93d6cdc80:/# apt-get update
 <Snip>

 apt-get install iputils-ping
 Reading package lists... Done
<Snip>

# Ping www.docker.com from within the container
root@6dd93d6cdc80:/# ping www.docker.com
PING www.docker.com (104.239.220.248) 56(84) bytes of data.
64 bytes from 104.239.220.248: icmp_seq=1 ttl=39 time=93.9 ms
64 bytes from 104.239.220.248: icmp_seq=2 ttl=39 time=93.8 ms
64 bytes from 104.239.220.248: icmp_seq=3 ttl=39 time=93.8 ms
^C
--- www.docker.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 93.878/93.895/93.928/0.251 ms
```

This shows that the new container can ping the internet and therefore has a valid and working network configuration.


# <a name="nat"></a>Step 4: Configure NAT for external connectivity

In this step we'll start a new **NGINX** container and map port 8080 on the Docker host to port 80 inside of the container. This means that traffic that hits the Docker host on port 8080 will be passed on to port 80 inside the container.

> **NOTE:** If you start a new container from the official NGINX image without specifying a command to run, the container will run a basic web server on port 80.

Start a new container based off the official NGINX image.

```
$ docker run --name web1 -d -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
386a066cd84a: Pull complete
7bdb4b002d7f: Pull complete
49b006ddea70: Pull complete
Digest: sha256:9038d5645fa5fcca445d12e1b8979c87f46ca42cfb17beb1e5e093785991a639
Status: Downloaded newer image for nginx:latest
b747d43fa277ec5da4e904b932db2a3fe4047991007c2d3649e3f0c615961038
```

Check that the container is running and view the port mapping.

```
$ docker ps
CONTAINER ID    IMAGE               COMMAND                  CREATED             STATUS              PORTS                           NAMES
b747d43fa277   nginx               "nginx -g 'daemon off"   3 seconds ago       Up 2 seconds        443/tcp, 0.0.0.0:8080->80/tcp   web1
6dd93d6cdc80        ubuntu              "sleep infinity"         About an hour ago   Up About an hour                                    reverent_dubinsky
```

There are two containers listed in the output above. The top line shows the new **web1** container running NGINX. Take note of the command the container is running as well as the port mapping - `0.0.0.0:8080->80/tcp` maps port 8080 on all host interfaces to port 80 inside the **web1** container. This port mapping is what effectively makes the containers web service accessible from external sources (via the Docker hosts IP address on port 8080).

Now that the container is running and mapped to a port on a host interface you can test connectivity to the NGINX web server.

To complete the following task you will need the IP address of your Docker host. This will need to be an IP address that you can reach (e.g. if your lab is in AWS this will need to be the instance's Public IP).

Point your web browser to the IP and port 8080 of your Docker host. The following example shows a web browser pointed to `52.213.169.69:8080`

![](concepts/img/browser.png)

If you try connecting to the same IP address on a different port number it will fail.

If for some reason you cannot open a session from a web broswer, you can connect from your Docker host using the `curl` command.

```
$ curl 127.0.0.1:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
    <Snip>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

If you try and curl the IP address on a different port number it will fail.

> **NOTE:** The port mapping is actually port address translation (PAT).

## <a name="Task #2"></a>Task #3

In this step you'll initialize a new Swarm, join a single worker node, and verify the operations worked.

1. Execute the following command on **node1**.

    ```
    node1$ docker swarm init
    Swarm initialized: current node (cw6jpk7pqfg0jkilff5hr8z42) is now a manager.
    To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3n2iuzpj8jynx0zd8axr0ouoagvy0o75uk5aqjrn0297j4uaz7-63eslya31oza2ob78b88zg5xe \
    172.31.34.123:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

2. Copy the entire `docker swarm join` command that is displayed as part of the output from the command.

3. Paste the copied command into the terminal of **node2**.

    ```
    node2$ docker swarm join \
    >     --token SWMTKN-1-3n2iuzpj8jynx0zd8axr0ouoagvy0o75uk5aqjrn0297j4uaz7-63eslya31oza2ob78b88zg5xe \
    >     172.31.34.123:2377

    This node joined a swarm as a worker.
    ```

4. Run a `docker node ls` on **node1** to verify that both nodes are part of the Swarm.

    ```
    node1$ docker node ls
    ID                           HOSTNAME          STATUS  AVAILABILITY  MANAGER STATUS
    4nb02fhvhy8sb0ygcvwya9skr    ip-172-31-43-74   Ready   Active
    cw6jpk7pqfg0jkilff5hr8z42 *  ip-172-31-34-123  Ready   Active        Leader
    ```

    The `ID` and `HOSTNAME` values may be different in your lab. The important thing to check is that both nodes have joined the Swarm and are *ready* and *active*.

# <a name="create_network"></a>Step 2: Create an overlay network

Now that you have a Swarm initialized it's time to create an **overlay** network.

1. Create a new overlay network called "overnet" by executing the following command on **node1**.

    ```
    node1$ docker network create -d overlay overnet
    0cihm9yiolp0s9kcczchqorhb
    ```

2. Use the `docker network ls` command to verify the network was created successfully.

    ```
    node1$ docker network ls
    NETWORK ID          NAME                DRIVER      SCOPE
    1befe23acd58        bridge              bridge      local
    0ea6066635df        docker_gwbridge     bridge      local
    726ead8f4e6b        host                host        local
    8eqnahrmp9lv        ingress             overlay     swarm
    ef4896538cc7        none                null        local
    0cihm9yiolp0        overnet             overlay     swarm
```

    The new "overnet" network is shown on the last line of the output above. Notice how it is associated with the **overlay** driver and is scoped to the entire Swarm.

    > **NOTE:** The other new networks (ingress and docker_gwbridge) were created automatically when the Swarm cluster was created.

3. Run the same `docker network ls` command from **node2**

    ```
    node2$ docker network ls
    NETWORK ID          NAME                DRIVER      SCOPE
    b76635120433        bridge              bridge      local
    ea13f975a254        docker_gwbridge     bridge      local
    73edc8c0cc70        host                host        local
    8eqnahrmp9lv        ingress             overlay     swarm
    c4fb141606ca        none                null        local
    ```

    Notice that the "overnet" network does not appear in the list. This is because Docker only extends overlay networks to hosts when they are needed. This is usually when a host runs a task from a service that is created on the network. We will see this shortly.

4. Use the `docker network inspect` command to view more detailed information about the "overnet" network. You will need to run this command from **node1**.

    ```
    node1$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "0cihm9yiolp0s9kcczchqorhb",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Containers": null,
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "257"
        },
        "Labels": null
    }
]
```

# <a name="create_service"></a>Step 3: Create a service

Now that you have a Swarm initialized and an overlay network, it's time to create a service that uses the network.

1. Execute the following command from **node1** to create a new service called *myservice* on the *overnet* network with two tasks/replicas.

    ```
    node1$ docker service create --name myservice \
    --network overnet \
    --replicas 2 \
    ubuntu sleep infinity

    e9xu03wsxhub3bij2tqyjey5t
    ```

2. Verify that the service is created and both replicas are up.

    ```
    node1$ docker service ls
    ID            NAME       REPLICAS  IMAGE   COMMAND
    e9xu03wsxhub  myservice  2/2       ubuntu  sleep infinity
    ```

    The `2/2` in the `REPLICAS` column shows that both tasks in the service are up and running.

3. Verify that a single task (replica) is running on each of the two nodes in the Swarm.

    ```
    node1$ docker service ps myservice
    ID            NAME         IMAGE   NODE   DESIRED STATE  CURRENT STATE  ERROR
    5t4wh...fsvz  myservice.1  ubuntu  node1  Running        Running 2 mins
    8d9b4...te27  myservice.2  ubuntu  node2  Running        Running 2 mins
    ```

    The `ID` and `NODE` values might be different in your output. The important thing to note is that each task/replica is running on a different node.

4. Now that **node2** is running a task on the "overnet" network it will be able to see the "overnet" network. Run the following command from **node2** to verify this.

    ```
    node2$ docker network ls
    NETWORK ID          NAME                DRIVER      SCOPE
    b76635120433        bridge              bridge      local
    ea13f975a254        docker_gwbridge     bridge      local
    73edc8c0cc70        host                host        local
    8eqnahrmp9lv        ingress             overlay     swarm
    c4fb141606ca        none                null        local
    0cihm9yiolp0        overnet             overlay     swarm
    ```

5. Run the following command on **node2** to get more detailed information about the "overnet" network and obtain the IP address of the task running on **node2**.

    ```
    node2$ docker network inspect overnet
    [
        {
            "Name": "overnet",
            "Id": "0cihm9yiolp0s9kcczchqorhb",
            "Scope": "swarm",
            "Driver": "overlay",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": [
                    {
                        "Subnet": "10.0.0.0/24",
                        "Gateway": "10.0.0.1"
                    }
                    ]
            },
            "Internal": false,
            "Containers": {
                "286d2e98c764...37f5870c868": {
                    "Name": "myservice.1.5t4wh7ngrzt9va3zlqxbmfsvz",
                    "EndpointID": "43590b5453a...4d641c0c913841d657",
                    "MacAddress": "02:42:0a:00:00:04",
                    "IPv4Address": "10.0.0.4/24",
                    "IPv6Address": ""
                }
            },      
            "Options": {
                "com.docker.network.driver.overlay.vxlanid_list": "257"
                },
                "Labels": {}
                }
            ]
    ```

You should note that as of Docker 1.12, `docker network inspect` only shows containers/tasks running on the local node. This means that `10.0.0.4` is the IPv4 address of the container running on **node2**. Make a note of this IP address for the next step (the IP address in your lab might be different than the one shown here in the lab guide).

# <a name="test"></a>Step 4: Test the network

To complete this step you will need the IP address of the service task running on **node2** that you saw in the previous step.

1. Execute the following commands from **node1**.

    ```
    node1$ docker network inspect overnet
    [
        {
            "Name": "overnet",
            "Id": "0cihm9yiolp0s9kcczchqorhb",
            "Scope": "swarm",
            "Driver": "overlay",
            "Containers": {
                "053abaa...e874f82d346c23a7a": {
                    "Name": "myservice.2.8d9b4i6vnm4hf6gdhxt40te27",
                    "EndpointID": "25d4d5...faf6abd60dba7ff9b5fff6",
                    "MacAddress": "02:42:0a:00:00:03",
                    "IPv4Address": "10.0.0.3/24",
                    "IPv6Address": ""
                }
            },      
            "Options": {
                "com.docker.network.driver.overlay.vxlanid_list": "257"
            },
            "Labels": {}
        }
    ]
    ```

    Notice that the IP address listed for the service task (container) running on **node1** is different to the IP address for the service task running on **node2**. Note also that they are one the sane "overnet" network.

2. Run a `docker ps` command to get the ID of the service task on **node1** so that you can log in to it in the next step.

    ```
    node1$ docker ps
    CONTAINER ID   IMAGE           COMMAND            CREATED      STATUS         NAMES
    053abaac4f93   ubuntu:latest   "sleep infinity"   19 mins ago  Up 19 mins     myservice.2.8d9b4i6vnm4hf6gdhxt40te27
    <Snip>
    ```

3. Log on to the service task. Be sure to use the container `ID` from your environment as it will be different from the example shown below.

    ```
    node1$ docker exec -it 053abaac4f93 /bin/bash
    root@053abaac4f93:/#
    ```

4. Install the ping command and ping the service task running on **node2**.

    ```
    root@053abaac4f93:/# apt-get update && apt-get install iputils-ping
    <Snip>
    root@053abaac4f93:/#
    root@053abaac4f93:/#
    root@053abaac4f93:/# ping 10.0.0.4
    PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.
    64 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=0.726 ms
    64 bytes from 10.0.0.4: icmp_seq=2 ttl=64 time=0.647 ms
    ^C
    --- 10.0.0.4 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 999ms
    rtt min/avg/max/mdev = 0.647/0.686/0.726/0.047 ms
    ```

    The output above shows that both tasks from the **myservice** service are on the same overlay network spanning both nodes and that they can use this network to communicate.

# <a name="discover"></a>Step 5: Test service discovery

Now that you have a working service using an overlay network, let's test service discovery.

If you are not still inside of the container on **node1**, log back into it with the `docker exec` command.

1. Run the following command form inside of the container on **node1**.

    ```
    root@053abaac4f93:/# cat /etc/resolv.conf
    search eu-west-1.compute.internal
    nameserver 127.0.0.11
    options ndots:0
    ```

    The value that we are interested in is the `nameserver 127.0.0.11`. This value sends all DNS queries from the container to an embedded DNS resolver running inside the container listening on 127.0.0.11:53. All Docker container run an embedded DNS server at this address.

    > **NOTE:** Some of the other values in your file may be different to those shown in this guide.

2. Try and ping the `myservice` name from within the container.

    ```
    root@053abaac4f93:/# ping myservice
    PING myservice (10.0.0.2) 56(84) bytes of data.
    64 bytes from ip-10-0-0-2.eu-west-1.compute.internal (10.0.0.2): icmp_seq=1 ttl=64 time=0.020 ms
    64 bytes from ip-10-0-0-2.eu-west-1.compute.internal (10.0.0.2): icmp_seq=2 ttl=64 time=0.041 ms
    64 bytes from ip-10-0-0-2.eu-west-1.compute.internal (10.0.0.2): icmp_seq=3 ttl=64 time=0.039 ms
    ^C
    --- myservice ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms
    rtt min/avg/max/mdev = 0.020/0.033/0.041/0.010 ms
    ```

    The output clearly shows that the container can ping the `myservice` service by name. Notice that the IP address returned is `10.0.0.2`. In the next few steps we'll verify that this address is the virtual IP (VIP) assigned to the `myservice` service.

3. Type the `exit` command to leave the `exec` container session and return to the shell prompt of your **node1** Docker host.

4. Inspect the configuration of the `myservice` service and verify that the VIP value matches the value returned by the previous `ping myservice` command.

    ```
    node1$ docker service inspect myservice
    [
        {
            "ID": "e9xu03wsxhub3bij2tqyjey5t",
            "Version": {
                "Index": 20
            },
            "CreatedAt": "2016-11-23T09:28:57.888561605Z",
            "UpdatedAt": "2016-11-23T09:28:57.890326642Z",
            "Spec": {
                "Name": "myservice",
                "TaskTemplate": {
                    "ContainerSpec": {
                        "Image": "ubuntu",
                        "Args": [
                            "sleep",
                            "infinity"
                        ]
                    },
    <Snip>
            "Endpoint": {
                "Spec": {
                    "Mode": "vip"
                },
                "VirtualIPs": [
                    {
                        "NetworkID": "0cihm9yiolp0s9kcczchqorhb",
                        "Addr": "10.0.0.2/24"
                    }
    <Snip>
    ```

    Towards the bottom of the output you will see the VIP of the service listed. The VIP in the output above is `10.0.0.2` but the value may be different in your setup. The important point to note is that the VIP listed here matches the value returned by the `ping myservice` command.

Feel free to create a new `docker exec` session to the service task (container) running on **node2** and perform the same `ping service` command. You will get a response form the same VIP.


## <a name="Task #2"></a>Task #4

1. Use a web browser to connect to the Login page of your UCP cluster

2. Enter your credentials as supplied by your lab instructor

3. Navigate to `Admin Settings` > `Routing Mesh` and enable the HTTP Routing Mesh (HRM) on port 80.

   ![](concepts/img/enable-hrm.png)

The HRM is now configured and ready to use.

# <a name="verify_hrm"></a>Step 2: Verify the HRM

Enabling the HRM creates a new *service* called `ucp-hrm` and a new network called `ucp-hrm`. In this step we'll confirm that both of these constructs have been created correctly.

Execute the following steps in the UCP web UI.

1. Navigate to `Resources` > `Networks` and check for the presence of the `ucp-hrm` network. You may have to `search` for it.

    ![](concepts/img/hrm-network.png)

    The network shows as an overlay network scoped to the entire Swarm cluster.

2. Navigate to `Resources` > `Services` and click the checkbox to `Show system services`.

    ![](concepts/img/hrm-svc1.png)

  The image above shows the `ucp-hrm` service up and running.

You have now verified that the HRM was configured successfully.

In the next two steps you'll create two services. Each service will based off the same `ehazlett/docker-demo:latest` image, and runs a web server that counts containers and requests. You will configure each service with a different number of tasks and each with a different value in the `TITLE` variable.

# <a name="create_red"></a>Step 3: Create the RED service

In this step you'll create a new service called **RED**, and configure it to use the HRM.

1. In `DDC` click `Resources` > `Services` and then `+Create Service`.

2. Configure the service as follows (leave all other options as default and remember to substitute "red.example.com" with the DNS name from your environment):
  - Name: `RED`
  - Image: `ehazlett/docker-demo:latest`
  - Scale: `10`
  - Arguments: `-close-conn`
  - Published port: Port = `8080/tcp`, Public Port = `5000`
  - Attached Networks: `ucp-hrm`
  - Labels: `com.docker.ucp.mesh.http` = `8080=http://red.example.com`
  - Environment Variables: `TITLE` = `RED`

  It will take a few minutes for this service to pull down the image and start.  Continue with the next step to create the **WHITE** service.

# <a name="create_white"></a>Step 4: Create the WHITE service

In this step you'll create a new service called **WHITE**. The service will be very similar to the **RED** service created in the previous step.

1. In `DDC` click `Resources` > `Services` and then `+Create Service`.

2. Configure the service as follows (leave all other options as default and remember to substitute "red.example.com" with the DNS name from your environment):
  - Name: `RWHITE`
  - Image: `ehazlett/docker-demo:latest`
  - Scale: `5`
  - Arguments: `-close-conn`
  - Published port: Port = `8080/tcp`, Public Port = `5001`
  - Attached Networks: `ucp-hrm`
  - Labels: `com.docker.ucp.mesh.http` = `8080=http://white.example.com`
  - Environment Variables: `TITLE` = `WHITE`

  This service will start instantaneously as the image is already pulled on every host in your UCP cluster.

3. Verify that both services are up and running by clicking `Resources` > `Services` and checking that both services are running as shown below.

  ![](concepts/img/check-svc.png)

You now have two services running. Both are connected to the `ucp-hrm` network and both have the `com.docker.ucp.mesh.http` label. The **RED** service is associated with HTTP requests for `red.example.com` and the **WHITE** service is associated with HTTP requests for `white.example.com`. This mapping of labels to URLs is leveraged by the `ucp-hrm` service which is published on port 80.


# <a name="test"></a>Step 5: Test the configuration

> **NOTE: DNS name resolution is required for this step. This can obviously be via the local hosts file, but this step will not work unless the URLs specified in the `com.docker.ucp.mesh.http` labels resolve to the UCP cluster nodes (probably via a load balancer).**

In this step you will use your web browser to issue HTTP requests to `red.example.com` and `white.example.com`. DNS name resolution is configured so that these URLs resolve to a load balancer which in turn balances requests across all nodes in the UCP cluster.

> Remember to substitute `example.com` with the domain supplied by your lab instructor.

1. Open a web browser tab and point it to `red.example.com`.

  ![](concepts/img/red.png)

  The text below the whale saying "RED" indicates that this request was answered by the **RED** service. This is because the `TITLE` environment variable for the **RED** service was configured to display "RED" here. You also know it is the **RED** service as this was the service configured with 10 replicas (containers).

2. Open another tab to `white.example.com`.

  ![](concepts/img/white.png)

  The output above shows that this request was routed to the **WHITE** service as it displays "WHITE" below the whale and only has 5 replicas (containers).

Congratulations. You configured two services in the same Swarm (UCP cluster) to respond to requests on port 80. Traffic to each service is routed based on the URL included in the `host` field of the HTTP header.

Requests arrive to the Swarm on port 80 and are forwarded to the `ucp-hrm` system service. The `ucp-hrm` service inspects the HTTP headers of requests and routes them to the service with the matching `com.docker.ucp.mesh.http` label.

## Wrap Up

Thank you for taking the time to complete this lab! Feel free to try any of the other labs. I
