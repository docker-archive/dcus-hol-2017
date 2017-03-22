# Lab X: Windows Docker Containers 101

Docker runs natively on Windows 10 and Windows Server 2016. In this lab you'll learn how to package Windows applications as Docker images and run them as Docker containers. You'll see how to organized distributed solutions using Docker Compose, and orchestrate them using Docker swarm mode.

> **Difficulty**: Beginner 

> **Time**: Approximately 30 minutes

> **Tasks**:
>
> * [Prerequisites](#prerequisites)
> * [Task 1: Run some simple Windows Docker containers](#task1)
>  * [Task 1.1: Run a task in a Nano Server container](#task1.1)
>  * [Task 1.2: Run an interactive Windows Server Core container](#task1.2)
>  * [Task 1.3: Run a background IIS web server container](#task1.3)
> * [Task 2: Package and run a custom app using Docker](#task2)
>  * [Task 2.1: Build a custom website image](#task2)
>  * [Task 2.2: Push you image to Docker Hub](#task2)
>  * [Task 2.3: Run your website in a container](#task2)
> * [Task 3: Run your custom app in a highly-available cluster](#task2)
>  * [Task 3.1: Create a Docker swarm with Windows Server nodes](#task2)
>  * [Task 3.2: Deploy your website as a service](#task2)
>  * [Task 3.3: Explore node management](#task2)

## Document conventions

When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value. 

For instance if you see `mstsc /v: <server1>` you would actually type something like `mstsc /v: server1-adaflds023asdf-23423kjl.appnet.com`

You will be asked to RDP into various nodes. These nodes are referred to as **v111node0**, **v111node1** etc. These tags correspond to the very beginning of the hostnames you will find in your welcome email. 

## <a name="prerequisites"></a>Prerequisites

You will be provided a set of Windows Server 2016 vritual machines running in Azure, which are already configured with Docker and the Windows base images. You do not need Docker running on your laptop, but you will need a Remote Desktop client to connect to the VMs. 

- Windows - use the built-in Remote Desktop Connection app.
- Mac - install [Microsoft Remote Desktop](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417?mt=12) from the app store.
- Linux - install [Remmina]http://www.remmina.org/wp/) (or any RDP client you prefer).

You will build images and push them to Docker Hub, so you can pull them on different Docker hosts. You will need a Docker ID.

- Sign up for a free Docker ID on [Docker Hub](https://hub.docker.com)

## <a name="task1"></a>Task 1: Run some simple Windows Docker containers

There are different ways to use containers:

1. In the background for long-running services like websites and databases
2. Interactively for connecting to the container like a remote server
3. To run a single task, which could be a PowerShell script or a custom app

In this section you'll try each of those options and see how Docker manages the workload. Before you start, let's check the Docker service is up and running. Open a PowerShell prompt from the taskbar shortcut, and run:

```
docker version
```

The Docker CLI connects to the Docker API running as a Windows Service, and you will see two sets of output. The first tells you the version and operating system where the Docker client is running, and the second tells you the version and operating system where the Docker service is running. Docker is cross-platform, so you can manage Linux hosts from Windows clients, and Windows hosts from Linux or Mac clients.

## <a name="task1.1"></a>Task 1.1: Run a task in a Nano Server container

This is the simplest kind of container to start with. In PowerShell run:

```
docker run microsoft/nanoserver powershell Write-Output Hello DockerCon 2017!
```

You'll see the output written from the PowerShell command. Here's what Docker did:

- created a new container from the [microsoft/nanoserver]() image, which is a basic Nano Server image, maintained by Microsoft
- started the container, running the `powershell` command and passing the `Write-Output...` arguments
- relayed the console output from the container to the PowerShell window

Docker keeps a container running as long as the process it starts is still running. In this case the `powershell` process completes when the `Write-Output` command finishes, so the container stops. The Docker platform is cautious with your data - it doesn't delete resources by default. You can list all the containers on the system and see the Nano Server container still exists, but is in the `Exited` state:

```
> docker ps --all
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS                      PORTS               NAMES
a525b7fb086b        microsoft/nanoserver   "powershell Write-..."   31 seconds ago      Exited (0) 19 seconds ago                       peaceful_benz
```

Containers which do one task and then end can be very useful. You could build a Docker image which installed the Azure PowerShell module and bundled a set of scripts you use to create a cloud deployment. Anyone can execute that task just by running the container - they don't need the scripts or the Azure module, they just need to pull the Docker image.


## <a name="task1.2"></a>Task 1.2: Run an interactive Windows Server Core container

Nano Server is a new operating system from Microsoft which supports a subset of the full Windows APIs. You can use it to package cross-platform applications like Java, Go, Node.js and .NET Core, but not full .NET Framework apps. For that there's the [microsoft/windowsservercore]() image which is a full Windows Server 2016 OS, without the UI.

You can explore an image by running an interactive container. Run this to start a Windows Server Core container and connect to it:

```
docker run -it --rm microsoft/windowsservercore powershell
```

- `-it` means run the container interactively, and connect a terminal session
- `--rm` means automatically remove the container when it exits

When the container starts you'll drop into a PowerShell session with the default prompt `PS C:\>`. This is PowerShell running inside a Windows Server Core container. Docker has attached to the console, relaying input and output between your PowerShell window and the PowerShell session in the container.

Run some commands to see how the Windows Server Core image is built:

- `ls c:\` - lists the C drive contents, you'll see there's a minimal installation of Windows 
- `Get-Process` - shows all running processes in the container. There are a number of Windows Services, and the PowerShell exe
- `Get-WindowsFeature` - shows the Windows feature which are available or already installed

You'll see that the base image has .NET 4.6 installed, which is backwards-compatible so you can run .NET 2.0 apps. Almost all Windows Server features are available (.NET 3.5 is an exception, but the [microsoft/aspnet:3.5]() image comes with that installed). You can install them with the `Add-WindowsFeature` cmdlet, which is how you would start to build up a custom application image from the base image, adding in the features you need.

Interactive containers are useful when you are putting together your own image. You can run a container and verify all the steps you need to deploy your app. Once you have it working, you can [export]() a running container to an image - but it's much better to capture those steps in a repeatable [Dockerfile](). You'll do that in a lter step.

Now run `exit` to leave the PowerShell session. That stops the container process, so the container exits. You ran the container with the `--rm` flag, so Docker will delete it - run `docker ps -a` now and you'll see only the original Nano Server container.


## <a name="task1.3"></a>Run a background IIS web server container

Background containers are how you'll run your application. Here's a simple exmaple using another image from Microsoft - [microsoft/iis]() which builds on top of the Windows Server Core image and installs the IIS Web Server feature:

```
docker run -d -p 80:80 --name iis microsoft/iis
```

- `-d` starts in detached mode, Docker runs the container in the background and monitors it
- `-p` publishes a port. The IIS image is built to allow traffic in on port 80. This maps port 80 on the host to port 80 in the container
- `--name` gives the container a name so you can refer to it when for management commands

Open a browser on your laptop and go to the address of your VM. You'll see the basic IIS homepage, which is being generated byu the container:

![IIS in a Windows Server Core container](images/iis.png)

Publishing the container port means any traffic the VM receives on port 80 gets forwarded to the IIS container, and the responses are forwarded back. Back in your Windows Server VM, run `docker ps` and you'll see the IIS container is still running. Run `Get-Process` and you'll see all the processes running in the VM, which will include a line for the IIS worker process, `w3wp` - something like this:

```
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
...
    187      19     4944      13096       0.09   4916   4 w3wp
```

It's important to realize that the container process - `w3wp.exe` in this case - is actually running on the host. That's what makes Docker containers so lightweight and so efficient, all containers are sharing the host's kernel. The container is using the filesystem built in the `microsoft/iis` image, which is where the `w3wp.exe` file lives. The Azure VM doesn't have IIS installed, everything the app needs is packaged in the image.

