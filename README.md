# DockerCon US 2017 Hands-On Labs (HOL)

![dcus2017](images/dockercon.png)

This repo contains the series of hands-on labs presented at DockerCon 2017. They are designed to help you gain experience in various Docker features, products, and solutions. Depending on your experience, each lab requires between 30-45 minutes to complete. They range in difficulty from easy to advanced.

You should be receiving an email with the necessary hostnames and credentials for the Azure VMs and Docker Datacenter clusters necessary to complete the labs. 

After you have your email, you can then choose one or more of the following lab tutorials to go through.

---

## [Continuous Integration With Docker Cloud](https://github.com/docker/dcus-hol-2017/tree/master/docker-cloud)

In this lab, you will learn how to configure a continuous integration (CI) pipeline for a web application using Docker Cloud's automated build features. You will complete the following tasks as part of the lab:

> **Difficulty**: Beginner
>
> **Time**: Approximately 20 minutes
>
> **Tasks / Concepts**
> 
> - Configure Docker Cloud to Automatically Build Docker Images
> - Configure Docker Cloud Autobuilds
> - Trigger an Autobuild

## [Docker Networking](https://github.com/docker/dcus-hol-2017/tree/master/docker-networking)

In this lab you will learn about key Docker Networking concepts. You will get your hands dirty by going through examples of a few basic networking concepts, learn about Bridge and Overlay networking, and finally learning about the Swarm Routing Mesh.

> **Difficulty**: Beginner to Intermediate
>
> **Time**: Approximately 45 minutes
>
> **Tasks / Concepts**
>
> * Networking Basics
> * Bridge Networking
> * Overlay Networking

## [Docker Orchestration](https://github.com/docker/dcus-hol-2017/tree/master/docker-orchestration)

In this lab you will play around with the container orchestration features of Docker. You will deploy a simple application to a single host and learn how that works. Then, you will configure Docker Swarm Mode, and learn to deploy the same simple application across multiple hosts. You will then see how to scale the application and move the workload across different hosts easily.

> **Difficulty**: Beginner
>
> **Time**: Approximately 30 minutes
>
> **Tasks / Concepts**
>
> * What is Orchestration
> * Configure Swarm Mode
> * Deploy applications across multiple hosts
> * Scale the application
> * Drain a node and reschedule the containers

## [Deploying Applications with Docker EE Advanced / Docker Datacenter](https://github.com/docker/dcus-hol-2017/tree/master/docker-enterprise)

In this lab you will deploy an application on Universal Control Plane (UCP) that takes advantage of some of the latest features of Docker Datacenter. Docker Datacenter is included with Docker EE Standard and Docker EE Advanced. The tutorial will lead you through building a compose file that can deploy a full application on UCP in one click. Capabilities that you will use in this application deployment include:

> **Difficulty**: Intermediate
>
> **Time**: Approximately 45 minutes
>
> **Tasks / Concepts**
> 
> * Docker services
> * Application scaling and failure mitigation
> * Layer 7 load balancing
> * Overlay networking
> * Application secrets
> * Application health checks
> * RBAC-based control and visibility with teams

## [Securing Apps with Docker EE Advanced / Docker Trusted Registry](https://github.com/docker/dcus-hol-2017/tree/master/securing-apps-docker-enterprise)

In this lab you will integrate Docker EE Advanced in to your development pipeline. You will build your application from a Dockerfile and push your image to the Docker Trusted Registry (DTR). DTR will scan your image for vulnerabilities so they can be fixed before your application is deployed. This helps you build more secure apps!


> **Difficulty**: Beginner
>
> **Time**: Approximately 30 minutes
>
> **Tasks / Concepts**:
>
> * Build a Docker Application
> * Pushing and Scanning Docker Images
> * Remediating Application Vulnerabilities

---

## Contribute Your Own Labs

If you have an awesome tutorial/lab and would like to add it here. Please open a PR. We would love to add more exciting tutorials to the list!
