---
date: "2020-06-09"
tags: ["smb", "circleci", "client"]
title: "Acceptance testing a SMB client on CircleCI"
---

## Background

The [hashicorp/go-getter](https://github.com/hashicorp/go-getter) library is largely used through HashiCorp own products such as [Packer](https://www.packer.io/), [Nomad](https://www.nomadproject.io/), and [Terraform](https://www.terraform.io/). According to the library documentation:    

> go-getter is a library for Go (golang) for downloading files or directories from various sources using a URL as the primary form of input. The power of this library is being flexible in being able to download from a number of different sources (file paths, Git, HTTP, Mercurial, etc.) using a single string as input. 

Thinking about the power of the library and to better respond to some of our products needs, [hashicorp/go-getter](https://github.com/hashicorp/go-getter) would implement a new Getter type to get artifacts from a shared folder using the SMB protocol. 

The proposed SMB getter would use the smbclient command and the code had to be well tested to make sure all the use cases were addressed. The challenge here was to start a SMB server on CircleCI and make the SMB getter tests, as the SMB client, try to download files from the shared folder.

### How SMB works

The SMB protocol works by allowing the communication between the SMB server and the SMB clients. 
You can start a SMB server in your machine with any desired configuration, such as authentication, and share the directory you like in the network you're connected. The other machines in the networks can access the shared directory, as SMB clients, whether they are permitted or not, by using the server IP or domain name. 

## How to run a SMB server and client on CircleCI

In order to have a SMB server/client communication working in place, it is necessary to have two instances in the same network that would represent each side of the communication, and docker containers make it possible to create that.

For the SMB server, the base image is the [dperson/samba](https://github.com/dperson/samba) image and for the client the `golang` base image, because [hashicorp/go-getter](https://github.com/hashicorp/go-getter) is implemented in golang. 

### Running multiple containers on CircleCI

It turns out that CircleCI allows to run docker commands, including docker-compose, to build and run docker images and container by just adding the step `setup_remote_docker` to the job steps. See [CircleCI documentation](https://circleci.com/docs/2.0/building-docker-images/) to learn more about it.


### Writing a docker-compose

To make sure the smb server and the go-getter containers are in the same network, the best way to this is to write a docker-compose file that will configure both containers. 

```
version: '3.2'

services:
  samba:
    build:
      dockerfile: Dockerfile-smbserver
      context: .
    container_name: samba
    environment:
      USERID: "0"
      GROUPID: "0"
    networks:
      - default
    ports:
      - "139:139/tcp" # default smb port
      - "445:445/tcp" # default smb port
    volumes:
      - /mnt:/mnt:z
    command: '-s "public;/public" -s "private;/private;yes;no;no;user" -u "user;password" -p'

  gogetter:
    build:
      context: .
    container_name: gogetter
    depends_on:
      - samba
    networks:
      - default
    volumes:
      - /mnt:/mnt
    command: tail -f /dev/null

networks:
  default:
```

This docker-compose creates the containers in the same network and makes sure the containers know each other by their names (samba and gogetter). In other words, the go-getter container can access any smb shared folder by using `samba` as a host name. 

The samba server will have a user, and two shares, one public and another private. The go-getter will depend on the samba container to start.

### Dockerfiles

Dockerfile:
```
# Dockerfile to create a go-getter container with smbclient dependency that is used by the get_smb.go tests
FROM golang:latest

COPY . /go-getter
WORKDIR /go-getter

RUN go mod download
RUN apt-get update
RUN apt-get -y install smbclient
```
A simple Dockerfile that will build an image with the necessary settings to run the SMB acceptance tests, including the smbclient, and it will be used to start the gogetter container. CircleCI doesn't allow to mount volumes and because of that it is necessary to copy the source code.


Dockerfile-smbserver:
```
# Dockerfile to create a smb server that is used by the get_smb.go tests
FROM dperson/samba

# Create shared folders
RUN mkdir -p /public && mkdir -p /private

# Create shared files and directories under the shared folders (data and mnt)
RUN echo 'Hello' > /public/file.txt && mkdir -p /public/subdir  && echo 'Hello' > /public/subdir/file.txt
RUN echo 'Hello' > /private/file.txt && mkdir -p /private/subdir  && echo 'Hello' > /private/subdir/file.txt
```
This creates the SMB server image with the mock files and directories in the public and private shares, and it will be used to start the samba container.

### The .circleci/config.yml file

With docker-compose and Dockerfiles set in the go-getter project, CircleCI job steps will build and start the containers using `docker-compose` command. If no version is set to `setup_remote_docker`, then CircleCI will use the default version `17.09.0-ce`.


```
 go-smb-test:
    docker:
      - image: circleci/golang:1.14.1
    steps:
      - checkout
      - setup_remote_docker

      - run:
          name: build and start smb server and gogetter containers
          command: |
            docker-compose build
            docker-compose up -d

      - run:
          name: wait for containers to start
          command: sleep 60

      - run:
          name: run smb getter tests
          command: |
            docker exec -it gogetter bash -c "env ACC_SMB_TEST=1 go test -v ./... -run=TestSmb_"

```

The samba container takes around 1 minute to start and the necessary command to run the tests are executed inside the gogetter container. The tests will run inside the container and CircleCI will print out all of the logs and will fail whenever the tests fails inside the gogetter container. 

### Call for action

This configuration uses linux machines for the server and the client. For [Packer](https://www.packer.io/), as example, most of the known users that uses SMB protocol to get artifacts are Windows users, therefore it will be great to make sure that the SMB getter is also tested for other OSes such as Windows and MacOS.

Besides that, there's always room for improvement and some future work can be added to list: 
- Speed up the image building by using the `docker_layer_caching` provided by CircleCI.
- Store test results

If you think you can contribute to making some improvements on this, HashiCorp would love to have your contribution either by opening a pull request with the solution or reporting an [issue](https://github.com/hashicorp/go-getter/issues) as a proposed enhancement to the [hashicorp/go-getter](https://github.com/hashicorp/go-getter) library. Feel welcome! 