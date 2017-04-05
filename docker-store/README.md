# Lab XX: Deploying Certified Content with Docker Store


> **Difficulty**: Beginner

> **Time**: Approximately 15 minutes

> In this lab you will use Docker store to find certified content. Then you'll deploy that content into your Azure instance. 

> - [Task 0: Configure the prerequisites](#prerequisits)
- [Task 1: Configure Docker Cloud to Automatically Build Docker Images](#deploy_app)
  - [Task 1.1: Configure Docker Cloud autobuilds](#autobuild)
  - [Task 1.2: Test autobuilds](#test_autobuild)


## What is Docker Store and Docker Certified content?
Docker store is your one stop to find a wide variety of Docker images and plugins. Of particular instance to IT practitioners are Docker Certified Images and plugins. Certified containers and plugins afford the highest degree of confidence. 

##Document conventions
When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value. 

For instance if you see `ssh <username>@<hostname>` you would actually type something like `ssh labuser@v111node0-adaflds023asdf-23423kjl.appnet.com`

You will be asked to SSH into various nodes. These nodes are referred to as **v111node0**, **v111node1** etc. These tags correspond to the very beginning of the hostnames you will find in your welcome email. 

## <a name="prerequisites"></a>Task 0: Prerequisites

In order to complete this lab, you will need the following:

- A Docker ID
- A Docker host (you can uone of the Azure nodes supplied in your registration email)


### Obtain a Docker ID

If you do not already have a Docker ID, you will need to create one now. Creating a Docker ID is free, and allows you to use both [Docker Cloud](https://cloud.docker.com) and [Docker Hub](https://hub.docker.com).

If you already have a Docker ID, skip to the first task.

To create a Docker ID:

1. Use your web browser to visit [`https://store.docker.com`](https://store.docker.com)

2. On the upper right hand side of the screen click **Log In**.

3. Near the bottom of the next screen click **Create Account**

4. Choose a username and password, and supply a valide email address. The click the box to signify you accept the terms of service and privacy policy. 

5. Click **Sign Up**

4. Check your email (**including your spam folder**) for a confirmatino email

5. Follow the steps outlined in the email.

6. You should be redirected back to `https://store.docker.com`

You now have a Docker ID. Remember to keep the password safe and secure.

# <a name="store_login"></a>Task 1: Login to Docker Store

1. If you're not already there, navigate to [https://store.docker.com](https://store.docker.com)

2. In the upper right hand corner click **Log In**



1. If you have not already SSH into one of the Docker hosts provided to you in the signup email. 

	ssh <user>@<Node DNS or IP>

