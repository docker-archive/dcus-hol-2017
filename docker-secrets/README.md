# Lab X: Docker Secrets

In this lab we are going to walk through what secrets are, how they work, and what you might use them for.

> **Difficulty**: Beginner

> **Time**: Approximately 10 minutes

> **Tasks**:
>
> * [Prerequisites](#prerequisites)
> * [Task 1](#task1)
> * [Task 2](#task2)
> * [Task 3](#task3)
> * [Task 4](#task4)
> * [Task 5](#task5)

## Document conventions

When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value. 

For instance if you see `ssh <username>@<hostname>` you would actually type something like `ssh labuser@v111node0-adaflds023asdf-23423kjl.appnet.com`

You will be asked to SSH into various nodes. These nodes are referred to as **v111node0**, **v111node1** etc. These tags correspond to the very beginning of the hostnames you will find in your welcome email. 

## <a name="prerequisites"></a>Prerequisites

- Lists all your prerequisites

## <a name="Task 1"></a>Task #1

Before we begin, we will need to:

1. Provide an overview of what will be done in this section
2. Use a bulleted list

## Getting started

Before we dive in, please make sure you are running Docker 17.03 (formerly known as 1.13) or higher, by running `docker version`.

```
$ docker version
Client:
 Version:      17.05.0-dev
 API version:  1.29
 Go version:   go1.7.5
 Git commit:   08544b1
 Built:        Sun Mar 26 11:58:19 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.05.0-dev
 API version:  1.29 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   08544b1
 Built:        Sun Mar 26 11:58:19 2017
 OS/Arch:      linux/amd64
 Experimental: true
```

Next, because secrets are only designed to work with Docker Swarm mode, you’ll need to enable it, by running `docker swarm init --advertise-addr $(hostname -i)`.

```
$ docker swarm init --advertise-addr $(hostname -i)
Swarm initialized: current node (espmasd8y3jd8h9gxjd1xhrvf) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-1gx02ztr66x99ecpaey3pxdv6re6hk5b86ozuam2zh80akiymt-6jsa9j6k3ybs5u
8421zmpbxjw \
    10.0.3.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

At this point, we are pretty much ready to play around with Docker Secrets, you can get a sense for how they function by running `docker secret` (to get the default help message). 

```
$ docker secret

Usage:  docker secret COMMAND

Manage Docker secrets

Options:
      --help   Print usage

Commands:
  create      Create a secret from a file or STDIN as content
  inspect     Display detailed information on one or more secrets
  ls          List secrets
  rm          Remove one or more secrets

Run 'docker secret COMMAND --help' for more information on a command.
```

As you can see, we can use the `docker secret` sub-command to `create`, `inspect`, `list`, and `remove` secrets. So, lets create three secrets, by running these `docker secret create` commands:

`echo "This is a secret API key" | docker secret create my_secret_apikey -`

`echo "This is a secret Redis password" | docker secret create my_secret_redispw -`

`echo "This is a secret SSL cert" | docker secret create my_secret_cert -`

You can see here, we are passing in our "secret" as a string via the echo command, then piping this as input into `docker secret create <your secret's name>`. After you do this, you should see something similar to this in your terminal.

```
$ echo "This is a secret API key" | docker secret create my_secret_apikey -
oxzjchqlhe2t5vyj7upyxmxdh

$ echo "This is a secret Redis password" | docker secret create my_secret_redispw -
59mycicjnhv34mg1m8eqi61ln

$ echo "This is a secret SSL cert" | docker secret create my_secret_cert -
dhvv6rw2e38r6nyjdigfpmxmb
```

Okay, so we created three secrets, now what? Well, can you use `docker secret ls` to list our newly created secrets.

```
$ docker secret ls
ID                          NAME                CREATED             UPDATED
59mycicjnhv34mg1m8eqi61ln   my_secret_redispw   9 seconds ago       9 seconds ago
dhvv6rw2e38r6nyjdigfpmxmb   my_secret_cert      4 seconds ago       4 seconds ago
oxzjchqlhe2t5vyj7upyxmxdh   my_secret_apikey    26 seconds ago      26 seconds ago
```





## <a name="Task #2"></a>Task #2

So, we now know how to create & list secrets. But, how do we actually use them inside a container? Well, lets create a service and attach our secret to it, by running `docker service create --name="redis" --secret="my_secret_redispw" redis:alpine` to see how secrets actually work in practice.

```
$ docker service create --name="redis" --secret="my_secret_redispw" redis:alpine
bckqr7zxbganntsbsunra3j2e
```

Next, lets verify our service was actually created, by running `docker service ls`.

```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE
bckqr7zxbgan        redis               replicated          1/1                 redis:alpine
```

Great, now that the service was created successfully, lets verify the process is in a "Running" state, by running `docker service ps redis`. If you don't see the state as "Running" yet, it might still be downloading, but that shouldn't take more than a minute or two.

```
$ docker service ps redis
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
ug9ts04gqta2        redis.1             redis:alpine        node1               Running             Running 59 seconds ago
```


## <a name="Task #3"></a>Task #3

So, at this point, you hopefully know what secrets are, how to create them, how to attach them to container, so lets explore what it actually looks like at an OS level by attaching to the container. Lets run `docker exec -it $(docker ps --filter name=redis -q) /bin/sh` to get a shell inside our redis continer.

```
$ docker exec -it $(docker ps --filter name=redis -q) /bin/sh
/data #
```

Now, lets change directories into /run/secrets by running `cd /run/secrets`.

```
/data # cd /run/secrets
/run/secrets #
```

Next, lets list the contents by running `ls`.

```
/run/secrets # ls
my_secret_dbpw
```

Finally, lets display the contents of "my_secret_dbpw" by running `cat my_secret_dbpw`.

```
/run/secrets # cat my_secret_dbpw
This is a secret DB password
```

Then disconnect from the container by running `exit`.

```
/run/secrets # exit
```


## <a name="Task #4"></a>Task #4

Lets create one more service and attach a different secret to it this time. Maybe we are running a series of secure web servers where the traffic is encrypted. So, we need to pass in our TLS certificates.

Again, you can list the existing services by running `docker service ls`. We already have our "redis" one here, but we will add an additional nginx one shortly.

```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE
zdiytovncaz6        redis               replicated          1/1                 redis:alpine
```

Don't forget you can also list secrets by running `docker secret ls`.

```
$ docker secret ls
ID                          NAME                CREATED             UPDATED
59mycicjnhv34mg1m8eqi61ln   my_secret_redispw   7 minutes ago       7 minutes ago
dhvv6rw2e38r6nyjdigfpmxmb   my_secret_cert      7 minutes ago       7 minutes ago
oxzjchqlhe2t5vyj7upyxmxdh   my_secret_apikey    8 minutes ago       8 minutes ago
```

So, lets create the new "nginx" service and attach our "my_secret_cert" secret, by running `docker service create --name="nginx" --secret="my_secret_cert" nginx:1.11.12`.

```
$ docker service create --name="nginx" --secret="my_secret_cert" nginx:1.11.12
zg4ase8jbl13ptqjqeyoclpai
```

Next, lets verify it was created as we expect by running `docker service ls`. You should now see something where both redis and nginx are listed.

```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE
zdiytovncaz6        redis               replicated          1/1                 redis:alpine
zg4ase8jbl13        nginx               replicated          0/1                 nginx:1.11.12
```

Just like before, lets verify the process is in a "Running" state, by running `docker service ps nginx`. If you don't see the state as "Running" yet, it might still be downloading, but that shouldn't take more than a minute or two.

```
$ docker service ps nginx
ID                  NAME                IMAGE               NODE                DESIRED STATE   CURRENT STATE            ERROR               PORTS
kk1rrxpxj25m        nginx.1             nginx:1.11.12       node1               Running   Running 39 seconds ago
```


## <a name="Task #5"></a>Task #5

Alright, now that the nginx service is up and running, lets attach a shell to it just like we did before, by running `docker exec -it $(docker ps --filter name=nginx -q) /bin/bash`.

```
$ docker exec -it $(docker ps --filter name=nginx -q) /bin/bash
root@2f8719bf0d76:/#
```

Now, lets change directories into /run/secrets by running `cd /run/secrets`.

```
root@2f8719bf0d76:/# cd /run/secrets/
root@2f8719bf0d76:/run/secrets#
```

Next, lets list the contents by running `ls`.

```
root@2f8719bf0d76:/run/secrets# ls
my_secret_cert
```

Finally, lets display the contents of "my_secret_cert" by running `cat my_secret_cert`.

```
root@2f8719bf0d76:/run/secrets# cat my_secret_cert
This is a secret SSL cert
```

Then disconnect from the container by running `exit`.

```
root@2f8719bf0d76:/run/secrets# exit
```


## Links



* [Manage sensitive data with Docker secrets](https://docs.docker.com/engine/swarm/secrets/)
* [docker secret](https://docs.docker.com/engine/reference/commandline/secret/)
* [Introducing Docker Secrets Management](https://blog.docker.com/2017/02/docker-secrets-management/)




## Wrap Up

Thank you for taking the time to complete this lab! Feel free to try any of the other labs.
