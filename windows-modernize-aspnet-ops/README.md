# Lab X: Modernize .NET Apps - for Ops

You'll already have a process for deploying ASP.NET apps, but it probably involves a lot of manual steps. Work like copying application content between servers, running interactive setup programs, modifying configuration items and manual smoke tests all add time and risk to deployments. 

In Docker, the process of packaging applications is completely automated, and the platform supports automatic update and rollback for application deployments. You can build Docker images from your existing application artifacts, and run ASP.NET apps in containers without changing code.

This lab is aimed at ops and system admins. It steps through packaging an ASP.NET WebForms app to run in a Docker container on Windows 10 or Windows Server 2016. It starts with an MSI and ends by showing you how to package the application from source. You'll see how easy it is to start running applications in Docker, and the benefits you get from a modern application platform.

## What You Will Learn

In this self-paced lab, you'll learn how to:

- Package an existing ASP.NET MSI so the app runs in Docker, without any application changes.

- Create an upgraded package with application updates and Windows patches.

- Update and rollback the running application in a production environment with zero downtime.

- Package an ASP.NET application from the source code without needing Visual Studio or MSBuild.

> **Difficulty**: Beginner 

> **Time**: Approximately 30 minutes

> **Tasks**:
>
> * [Prerequisites](#prerequisites)
> * [Task 1: Packaging ASP.NET Apps as Docker Images](#task1)
>   * [Task 1.1: Installing MSIs in Docker images](#task1.1)
>   * [Task 1.2: Building the v1.0 Application Image](#task1.2)
>   * [Task 1.3: Running the Application v1.0 in Docker](#task1.3)
> * [Task 2: Upgrading Application Images](#task2)
>   * [Task 2.1: Updating Windows and app versions](#task2.1)
>   * [Task 2.2: Building the v1.1 Application Image](#task2.2)
>   * [Task 2.3: Upgrading the Running Application](#task2.3)
> * [Task 3: Zero-Downtime Update and Rollback](#task3)
>   * [Task 3.1: Create a Docker swarm with Windows Server nodes](#task3.1)
>   * [Task 3.2: Deploy your website as a service](#task3.2)
>   * [Task 3.3: Take a node offline for maintenance](#task3.3)
> * [Task 4: Packaging Applications From Source](#task4)
>   * [Task 3.1: Create a Docker swarm with Windows Server nodes](#task3.1)
>   * [Task 3.2: Deploy your website as a service](#task3.2)
>   * [Task 3.3: Take a node offline for maintenance](#task3.3)

## Document conventions

When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value. 

For instance if you see `$ip = <ip-address>` you would actually type something like `$ip = '10.0.0.4'`

You will be asked to RDP into various servers. You will find the actual server names to use in your welcome email. 

## <a name="prerequisites"></a>Prerequisites

You will be provided a set of Windows Server 2016 virtual machines running in Azure, which are already configured with Docker and the Windows base images. You do not need Docker running on your laptop, but you will need a Remote Desktop client to connect to the VMs. 

- Windows - use the built-in Remote Desktop Connection app.
- Mac - install [Microsoft Remote Desktop](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417?mt=12) from the app store.
- Linux - install [Remmina](http://www.remmina.org/wp/), or any RDP client you prefer.

You will build images and push them to Docker Hub, so you can pull them on different Docker hosts. You will need a Docker ID.

- Sign up for a free Docker ID on [Docker Hub](https://hub.docker.com)

## <a name="task1"></a>Task 1: Packaging ASP.NET Apps as Docker Images

A Docker image packages your application and all the dependencies it needs to run into one unit. Microsoft maintain the [microsoft/aspnet](https://hub.docker.com/r/microsoft/aspnet/) image on Docker Hub, which you can use as the basis for your own application images. It is based on [microsoft/windowsservercore](https://hub.docker.com/r/microsoft/windowsservercore/) and has IIS and ASP.NET already installed. 

In this lab you'll start with an MSI that deploys a web app onto a server and expects IIS and ASP.NET to be configured. If you already have a scripted build process then you may have MSIs or Web Deploy packages already being generated, and it's easy to package them into a Docker image.

## <a name="task1.1"></a> Task 1.1: Installing MSIs in Docker images

Building a Docker image from an MSI is very easy, you just need to copy in the `msi` file and run `msiexec` to install it. Have a look at the [Dockerfile](v1.0/Dockerfile) for the app you're going to deploy and later upgrade.

> If you noticed the version of [microsoft/windowsservercore](https://hub.docker.com/r/microsoft/windowsservercore/) is an old one - that's deliberate. You'll be updating the Windows version as well as the app version in this lab.

There are just three lines of script in the simple [Dockerfile syntax](https://docs.docker.com/engine/reference/builder/):

- `FROM` specifies the base image to use as a starting point, in this case a specific version of the ASP.NET image
- `COPY` copies the existing v1.0 application MSI from the local machine into the Docker image
- `RUN` installs the MSI using `msiexec`, with the `qn` switch to install silently, and passing a value to the custom `RELEASENAME` variable that the MSI uses

## <a name="task1.2"></a> Task 1.2: Building the v1.0 Application Image

Every Docker image has a unique name, and you can also tag images with additional information like application version numbers. To build the image, RDP into one of your Azure VMs, open a PowerShell prompt from the taskbar shortcut, and run:

```
cd C:\scm\github\docker\dcus-hol-2017\windows-modernize-aspnet-ops\v1.0
docker build -t <DockerID>/modernize-aspnet-ops:1.0 .
```

> Be sure to tag the image with your own Docker ID - you'll be pushing it to Docker Hub next.

The output from `docker build` shows you the Docker engine executing all the steps in the Dockerfile. On your lab VM the base images have already been pulled, so the image should build quickly.

When the build completes you'll have a new image stored locally, with the name `<DockerId>/modernize-aspnet-ops` and the tag `1.0` indicating that this is version 1.0 of the app.

Login to Docker Hub with your Docker ID and push that image, so it's available for the other VMs you'll use later on:

```
docker login
...
docker push <DockerID>/modernize-aspnet-ops:1.0
```

## <a name="task1.3"></a> Task 1.3: Running the Application v1.0 in Docker

The sample application for the lab is a simple ASP.NET WebForms app, which the MSI installs to the default IIS website running on port 80. To start the application, use `docker run` and publish the port from the container onto the host, so the website is available externally:

```
docker run -d -p 80:80 --name v1.0 <DockerID>/modernize-aspnet-ops:1.0
```

- `-d` starts the container in detached mode, so Docker keeps it running in the background
- `-p` publishes port 80 on the container to port 80 on the host, so Docker directs incoming traffic into the container
- `--name` gives the container the name `v1.0`, so you can refer to it in other Docker commands

`docker ps` will show you that the container is running, together with the port mapping and the command running inside the container:

```
> docker ps
CONTAINER ID        IMAGE                                    COMMAND                   CREATED             STATUS                    PORTS                NAMES
1ab0fa9228b8        dockersamples/modernize-aspnet-ops:1.0   "C:\\ServiceMonitor..."   17 seconds ago      Up 1 second               0.0.0.0:80->80/tcp   v1.0
```

You can get basic management information about the container with `docker top v1.0` to list the running processes, and `docker logs v1.0` to view application log entries. In this case, IIS doesn't write any logs to the console so there won't be any log entries surfaced to Docker.

Open a browser on your laptop and go to the address of your VM. You'll see the sample app, which just shows some basic diagnostics details:

![Version 1.0 of the sample app](img/app-v1.0.png)

## <a name="task2"></a>Task 2: Upgrading Application Images

The Docker image from [Part 1](part-1.md) is a snapshot of the application, built with a specific version of the app and a specific version of Windows. When you have an upgrade to the app or an operating system update you don't make changes to the running container - you build a new Docker image which packages the updated components and replace the container with a new one. 

Microsoft are releasing [monthly updates to the Windows base images](https://hub.docker.com/r/microsoft/windowsservercore/tags/) on Docker Hub. When your applications are running in Docker containers, there is no 'Patch Tuesday' with manual or semi-automated update processes. The Docker build process is fully automated, so when a new version of the base image is released with security patches, you just need to rebuild your own images and replace the running containers.

## <a name="task2"></a>Task 2.1: Updating Windows and app versions

Take a look at the [updated Dockerfile](v1.1/Dockerfile) for version 1.1 of the application. It's the same structure as 1.0 but with some important changes:

- the `FROM` image is tagged with version `10.0.14393.693`, this is the latest Windows image. v1.0 was built on Windows version `10.0.14393.576`
- the MSI is version `1.1.0.0` which contains an updated application release
- the MSI parameter `RELEASENAME` has been changed to `2017.03` - this value gets shown in the web app

The Dockerfile is still a simple 3-line script. All that's changed are the version details for Windows and the application. Microsoft are releasing new versions of the Windows base image monthly, which contain all the latest security patches and hotfixes.

Version 1.1 represents a change to the app which coincides with an updated Windows version being available, so it packages both application and OS updates.

## <a name="task2"></a>Task 2.2: Building the v1.1 Application Image

The process to build the new version is identical. In PowerShell, switch to the 1.1 directory and run `docker build`, using a new tag to identify the version:

```
cd ..\v1.1
docker build -t <DockerId>/modernize-aspnet-ops:1.1 .
```

Now you have two application images, tagged `1.0` and `1.1`, each containing different versions of the application built on different versions of Windows. You can see the basic image details with the `docker image ls` command:

```
> docker image ls --filter reference='*/modernize*'
REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
sixeyed/modernize-aspnet-ops           1.1                 dcea5c0e1be9        41 minutes ago      10.1 GB
sixeyed/modernize-aspnet-ops           1.0                 e763f76db517        About an hour ago   10 GB
```

You can see the images are listed at around 10GB each, but this is the logical size. Physically, the images share the majority of data in read-only image layers. There are a lot more images on your lab VM - run `docker image ls` to see them all, and then run `docker system df` to see how much physical storage they are using.

You'll see there are around 20 images, with logical sizes totalling over 170GB - but the actual storage used is around 20GB.

## Upgrading the Running Application

Version 1.0 of the application is still running, and the container port is mapped to port 80 on the host. Only one process can listen on a port, so you can't start a new container which also listens on port 80. 

You can test out the new version locally by running a new container and publishing to port 8081:

```
docker run -d -p 8081:80 --name v1.1 dockersamples/modernize-aspnet-ops:1.1
```

On the Azure VM, port 8081 isn't open so no-one can see the site externally. To browse it on the VM, you need to find the container's IP address by running:

```
docker inspect --format '{{ .NetworkSettings.Networks.nat.IPAddress }}' v1.1
```

You can open Firefox on the VM and browse to *http://<container-ip-address>:8081* to see the updated version of the app:

![Version 1.1 of the sample app](img/app-v1.1.png)

The new website shows the updated application version, which is read from the app DLL, and the release version number, which is read from the MSI parameter. The colors have changed too, to make the versions stand out when you're running side-by-side. 

In non-production environments you can upgrade just by killing the old container and starting a new one, using the new image but mapping to the original port. That's a manual approach which will incur a few seconds downtime. Docker provides an automated, zero-downtime alternative which you'll use instead.

