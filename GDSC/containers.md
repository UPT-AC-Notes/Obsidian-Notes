Containers
==========

Intro
-----

In the current day, we're surrounded by applications, and more specifically, by web-based applications, and many businesses are based on them. But what is the best solution to deploy, host, scale, maintain, update and upgrade them?

An application ultimately requires some CPU, memory, storage, and networking resources. The way those resources have been consumed and how they were scaled evolved over time:

**Traditional deployment era**:
- Running applications on physical servers.
- No way to define boundaries for applications, causing other applications to underperform. This could be addressed by running multiple applications on multiple servers, but resources could be underutilized, and it can be difficult to maintain so many physical nodes. (later fixed by resource limits and cgroups).
- Potential downtimes if a server goes down, or if it needs to be set in maintenance mode for updates and upgrades.
- Long time to install from scratch.
- Dependency versions conflict between applications (e.g.: python packages).

**Virtualized deployment era**:
- In response to this issue, VMs started being used, allowing applications to be isolated between VMs, providing a higher level of security because the application data cannot be freely accessed by another application.
- Offers better utilization of resources in a physical server, new VMs containing new application instances can be created on demand.
- In addition to that, VMs can offer a plaethora of useful features:
    - Can be paused / suspended / resumed.
    - Considerably easier to create new VMs and have an OS and application up and ready due to sysprepped image templates / golden images. This is further improved by software like ``cloud-init`` (Linux VMs) or ``cloudbase-init`` (Windows VMs), which allows users to pass in their own userdata scripts, which will be executed on the VM's first boot. This significantly reduces the amount of time needed to spin up new VMs with a fresh OS installed from 15-45 minutes to ~1 minute, including the initial boot.
    - Can be cold migrated or even live migrated, which can be useful when a node has to go into maintenance mode and the VMs need to be evacuated first (live migration: the ability to completely migrate / move a VM while it is still running, without any interruption on the VM side).
    - Can be snapshotted, allowing the state of the VM to be saved and restored later.
    - Can have memory ballooning / Dynamic Memory, meaning that the memory allocated to the VM expands as needed. This allows cloud providers to overcommit their available memory, with the assumption that not all the VMs will need **ALL** their allocated memory at the same time.
    - Storage and vCPUs can be overcommited as well.
    - Can be resized (vertical scaling), adding more memory, vCPUs, storage, disks, NICs, volumes, and other PCI devices. In some cases, these resources can be live resized, and some devices can be hotplugged.
    - Can failover VMs to other nodes in the cluster, meaning that if a node goes down, its VMs can be restarted on a different node. Special storage considerations need to be taken into account for this.
    - Resource usage (CPU utilization, memory usage, disk IOPS) can be monitored and tracked by the host / hypervisor, without needing any monitoring software inside the VMs.
    - QoS (Quality of Service) can be applied to different VM resources, throttling them.
    - Security Groups / Network Policies can be applied to the VMs themselves, limiting inbound and outbound traffic to specific rules that the user applies. These rules cannot be bypassed by the VM guests.
- Because it became so easy to simply create and destroy VMs on demand automatically, it became easy to automatically scale the number of VMs associated with an application (horizontal scaling), based on the tracked VM resource usage. The idea of "Pets vs Cattle" became more common; the application instances could now be treated as "Cattle" more easily (easy to scale; if one "Cattle" dies, it doesn't matter because it'll be replaced soon by another).
- It's important to note that each VM has an entire operating system inside of it, each consuming CPUs cycles keeping its system processes in order. This can be considered a downside, since in most scenarios we only care about the applications being deployed and running inside the VM, anything else being considered an overhead.

**Container deployment era**:
- In response to this issue, containers have been created. They are more lightweight by sharing the OS and kernel with other applications and containers, allowing them to be more densely packed. By sharing the OS, the isolation properties are more relaxed when compared to VMs.
- By sharing the OS, they're not as flexible when it comes to supporting multiple OSes. For example, typically you can only have Linux containers on Linux and Windows containers on Windows (though that is subject to change thanks to Hyper-V isolated containers). There are additional restrictions when it comes to the Kernel version disparity between the host OS and the container image OS, which is especially true on Windows (which can be mitigated slightly through Hyper-V isolated containers).
- In exchange to all this, they provide a couple of useful benefits:
    - They are more efficient when compared to VMs, allowing for a higher density of applications deployed.
    - When compared to VM images, Container images are more lightweight, easier to build and easier to use. In addition to this, continuous development, integration, and deployment is possible with frequent container image builds, allowing users to also rollback due to image immutability.
    - Increased observability, including being able to monitors application health and other signals.
- When it comes to security, there are some container runtimes that offer increased security, such as Kata Containers, or Hyper-V isolated containers. It may be useful to consider them in some situations; over the years, there were plenty of security issues / vulnerabilities in containers, allowing attackers to escape the container context and exposing host system resources and data, possibly with escalated privileges.

**Future era?**:
- This section is a bit speculative, as we cannot predict the future, but we can observe some trends and see what people are focusing on.
- WebAssembly (WASM):
    - One such technology is called WebAssembly, which is a binary instruction format for a stack-based VM. Originally, It was designed to bring near-native performance to web applications using **any** programming language and to be able to run on any OS. It also boasts a few other qualities, such as a memory-safe, sandboxed execution environment, open and debuggable.
    - Interestingly, WebAssembly doesn't have any Web-specific assumptions, it's runtime environment can easily be embedded into host applications. Thus, WebAssembly System Interface (WASI) was designed as a simple ABI and API, providing POSIX-like features, which can then allow for software containerization. One of the founders of Docker even said: "If WASM+WASI existed in 2008, we wouldn't have needed to create Docker. That's how important it is. WebAssembly on the server is the future of computing."
    - One of the most recent developments in this area is that Containerized WASM Applications can now be deployed using Kubernetes. These applications are first built into an OCI image, and then they are basically deployed almost in the same manner as other Kubernetes Pods, the only main difference is that they require the ``runtimeClassName`` field to be set to be a WASM runtime. Some of its benefits are that the OCI images are smaller (especially when compared to Windows images) and that the OCI images are cross-platform, meaning that the same OCI image can be used on any OS or CPU architecture.
    - There might be some limitations for certain WebAssembly platforms when it comes to application development. For example, the Python documentation states that the ``threading`` and ``multiprocessing`` modules are not supported on the ``wasm32-emscripten`` and ``wasm32-wasi`` platforms.
- Unikernels:
    - Unikernels are a different application deployment model. They are meant to be almost as lightweight and resource-friendly as Containers, but each unikernel having its own kernel, reducing the attack surface and increasing security.
    - The Images have baked into them a lightweight kernel, known as a library operating system (which is not a full-fledged OS like Linux). This kernel would only support a single user, a single address space, and a single process. Additionally, the image building process is not as simple as building the Container images.


What is a container
-------------------

How containers came into existence:

- Developer: Well, this application works on my machine.
- Boss: So what, you're expecting us to deliver your machine?
- Developer: Yes.

Consider this: most applications may require specific dependencies and packages to be installed. Having them run of different machines would require that machine to have the same dependencies and packages installed (+ installing them may require a substantial amount of time). However, based on the Operating System or Distro, they may not be available, or available at the same version (managing dependencies and versions is hell as well, making sure that there are no conflicting versions). Making sure that your application runs everywhere and anywhere is difficult to test and guarantee. However, what if we just bundle the application and its dependencies, and just ship that bundle to the machine we're trying to run the application? Well, that's what a container is.

As mentioned, containers are basically a method of packaging, deploying, and managing applications. They're a lot more lightweight than VMs, simply because they didn't have to pack an entirely new Operating System in the container image, it only contains what it should. Some container images are even smaller, only containing the executable it's supposed to run (images starting from scratch, scratch-based images), or are based on a minimal container image base, like alpine.

Additionally, container images can be versioned. Because of this, they also serve as a time capsule of an application.


Installing docker
-----------------

Docker is a common way of running containers, but keep in mind that Docker is more of an interface, rather than a container runtime itself. Docker uses `containerd` as a container runtime, meaning that docker will basically tell `containerd` what exactly to run. So, if you're asked if you're running Docker or `containerd`, the answer is Yes.

To install Docker, we can simply follow the official guide: https://docs.docker.com/engine/install/ubuntu/

After we've installed it, we can check its version and that it does actually use `containerd` as a runtime by running:

```
docker version
```

The output may be something like this:

```
Client: Docker Engine - Community
 Version:           24.0.5
 API version:       1.43
 Go version:        go1.20.6
 Git commit:        ced0996
 Built:             Fri Jul 21 20:35:23 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.5
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.6
  Git commit:       a61e2b4
  Built:            Fri Jul 21 20:35:23 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.22
  GitCommit:        8165feabfdfe38c65b599c4993d227328c231fca
 runc:
  Version:          1.1.8
  GitCommit:        v1.1.8-0-g82f18fe
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

As mentioned, docker is using `containerd`, which in turn is going to use `runc`. Keep in mind that there are other container runtimes with different focuses, such as the one for Kata Containers (more security focused), but this is a more wide spread runtime. They can be used interchangibly, since they all have to respect the same CRI (Container Runtime Interface).


Running containers
------------------

Containers are meant to be lightweight and small. This means that they won't have any GUI or desktop, we'll have to use commands in a terminal because of this. :)

Let's start our first container:

```
docker run hello-world
```

Output:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

...
```

So, what did we actually run? From what we can see in the messages above, it seems that we do have the docker client installed, which contacted the local docker server which is running, and the server pulled the `hello-world` container image and started a container from it. We can see the status of the container by running:

```
docker ps -a
```

Output:

```
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                      PORTS     NAMES
10d40b394cdb   hello-world   "/hello"   3 minutes ago   Exited (0) 3 minutes ago              zen_allen
```

As we can see from the output above, there is a container which started from the `hello-world` image, and has a Container ID and name associated with it. IDs are unique, but names may not. However, we can associate names with containers (``docker run --name sobolan-container hello-world``). The names are only meant to easily refer to them.

There are a few other noteworthy columns above: the Status `Exited (0) 3 minutes ago`, which basically means that the container is shut down and that it finished its process with an 0 exit code, which signals success (remember that `return 0;` you kept writing in `main` in your programming classes? That's the same 0). Regarding the `Status`, there are 2 types of containers, based on the 2 types of applications that we may have:

- containers in which we're running finite commands or applications, with the expectation that it is going to finish its job eventually. Typically used for compiling / building binaries, running tests, etc.
- containers in which we're running services ("infinite" duration commands / applications). Includes containers for running http servers, database servers, etc.

And another important field is the `Command` field. That basically says what command was run in the container, and it's typically composed from an `entrypoint` and `arguments`. It's basically the same thing as a commandline command, in which you could consider the first part as the "function name" and the rest as its arguments. But where did that command come from anyways? Well, that command came from the person that created and published the `hello-world` image. But it is something that we can override, if we ever need it:

```
docker run --entrypoint bash hello-world
docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "bash": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container:
```

We tried to run `bash` instead of `/hello` in the container, but we encountered an error: `"bash": executable file not found in $PATH`. This basically means that that command does not exist in the container image, or at least it cannot be found in the `$PATH` list of paths to check for executables. In this case, bash simply doesn't exist in the container image. Sometimes, image creators want the images to be as small as possible, so they do not include things which are not necessary for the container to do its job. In this case, the container image does not have anything other than the `/hello` binary. We can even check this by listing the locally downloaded images:

```
docker images -a
```

Output:

```
REPOSITORY    TAG      IMAGE ID       CREATED         SIZE
hello-world   latest   d2c94e258dcb   11 months ago   13.3kB
```

The image has 13.3KB. Not much to fit in there. Let's try pulling another image:

```
docker pull alpine:3.19
```

Alpine is a popular container image, it contains the minimal tooling necessary to run most commands and install most of the needed packages through `apk`. Note that we specified the version `3.19`. Docker images are typically versioned through this image tag, and they typically follow the SemVer (Semantic Versioning) versioning system. If you see a version like this `x.y.z` in your libraries or APIs, it can be explained like so:

- `x`: major version. You can expect major changes from one major version to another, and if you upgrade your libraries' major version, you should expect things to break, as they typically include backwards-incompatible changes: modules / functions / classes / methods may have been moved / removed / renamed, functions / methods may have different signatures, etc.
- `y`: minor version. Typically contains backwards-compatible changes, mostly additions and new features that do not impact the way consumers are currently using it. It should be safe to upgrade minor versions.
- `z`: revision number. Typically contains minor changes and / or bug fixes. It does not add or remove any existing behaviour.

SemVer versions may also have some suffixes, like `aN`, `bN`, `rcN`, which stands for the N-th `alpha`, `beta`, `release candidate` versions. They're not used by default, mostly for testing purposes before the actual "gold" release.

Going back to the `alpine` image, we can see that it is pretty small:

```
docker inspect alpine:3.19
```

Output:

```
[
    {
        "Id": "sha256:05455a08881ea9cf0e752bc48e61bbd71a34c029bb13df01e40e3e70e0d007bd",
        "RepoTags": [
            "alpine:3.19"
        ],
        ...
        "Config": {
            ...
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh"
            ],
            ...
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 7377074,
        ...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:d4fc045c9e3a848011de66f34b81f052d4f2c15a17bb196d637e526349601820"
            ]
        },
        ...
    }
]
```

Through this command, we can see its size (~7.4MB), and other details, like environment variables, command, entrypoint, etc. There are a few very important details here, like the `"Architecture"` and `"Os"` fields. In this case, we see that this image is destined for our host specifically, since we have a `linux/amd64` host (yours may be different).

Going back to the idea that containers share the same OS / kernel, it's important for a container image to **exactly** match our hosts. We can't run Windows containers on Linux, or vice-versa, or at least not in normal circumstances (for example, through Hyper-V Isolation, we could actually have Linux Containers on Windows, known as LCOWs). We can't run `linux/arm64` containers on a `linux/amd64` host, or at least not in normal circumstances (`arm64` can be emulated though through QEMU, but of course, there's a performance penalty because of emulation). If we want our application to run on **any OS** and **any platform** (CPU architecture), **we have to build** our binaries for all the `OS/platform` combination, and then package them into containers destined towards that specific `OS/platform` combination. Some programming languages may have it a bit easier though by not having to compile the code (interpreted languages like Python), but we'd still have to package the code in different combinations of `OS/platform` container images.

Let's try starting a container using this image:

```
docker run alpine:3.19
```

Nothing really happened. If we check its status, we see that it already exited:

```
docker ps -a
CONTAINER ID   IMAGE         COMMAND             CREATED             STATUS                         PORTS     NAMES
a3258fec88a8   alpine:3.19   "/bin/sh"           2 seconds ago       Exited (0) 2 seconds ago                 infallible_varahamihira
```

That happened because it ran its configured command and exited. It doesn't run any long-running service. We could instead make it do something that takes a bit more time:

```
docker run --name sobolan --rm alpine:3.19 sleep 600
```

We told it to sleep for 600 seconds. Now, if we'd do ``docker ps -a``, we'd see it is still `RUNNING`. In another terminal, we can even connect to it through `docker exec -ti sh`:

```
docker exec -ti sh
```

Now we're connected inside the container itself. We could have simply executed one single command without attaching to the contaienr through ``docker exec``. But since we're here, we can run whatever commands we need. For example, if we run `ps -aux`, we'll see:

```
# ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sleep 600
    7 root      0:00 sh
   13 root      0:00 ps aux
```

PID 1 is called the "init process", and it's essentially the "main" process of the container. If it dies, the container dies as well. Because the `sleep 600` command will end in 10 minutes, the container will die in 10 minutes. This PID 1 is not unique to containers, it's actually a basic Operating System concept, and we can see it on our Linux machines as well (after we run `exit` inside our containers):

```
ubuntu@ubuntu:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.1  0.1 170888 13176 ?        Ss   Mar25  51:17 /sbin/init
...
```

Though, if we're looking at processes running on our systems, let's run this command as well:

```
ps aux | grep "sleep 600"
```

Output:

```
 ps aux | grep sleep
ubuntu   3510446  0.2  0.2 1474992 23824 pts/4   Sl+  13:55   0:00 docker run --name sobolan --rm alpine:3.19 sleep 600
root     3510504  0.2  0.0   1604     4 ?        Ss   13:55   0:00 sleep 600
ubuntu   3510861  0.0  0.0   6432   660 pts/0    S+   13:55   0:00 grep --color=auto sleep
```

Interestingly, we can see both the `docker run` command and the actual `sleep 600` commands. But what happens when we kill that PID?

```
sudo kill -9 3510504
```

We see that the container actually dies. That's because that PID actually refers to PID 1 inside the container itself, and as we've mentioned, if that dies, the container dies. But as you've noticed, we're able to see the processes inside the container. That's called a process-isolated container. There are other types of containers as well, such as Kata containers or Hyper-V isolated containers, which offer a higher level of isolation and security. In those cases, we wouldn't be able to see the processes running inside the container.

Keep in mind that if we want the containers to keep living, we have to make sure that its PID 1 doesn't die.

But where are these images from? Well, they come from a container registry. Technically, the full names of the images we've used are actually `docker.io/hello-world` and `docker.io/alpine:3.19`. `docker.io` is the DockerHub registry, but there are other registries as well, and we could also host our own registries if we want. Note that this is a public registry, and there are some rate limits for free users, meaning that if you have a limited number of image pulls you can do per day. But for now, let's look at https://hub.docker.com.

Here we see all sorts of images, privately build, official images, published by verified publishers and so on. The general recommendation is to use official or verified images. For those images, we can also typically see a guide on how to use them. Let's take for example the nginx container image: https://hub.docker.com/_/nginx . In here, We can see how to use it:

```
docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d nginx
```

We already recognize some of the arguments here. There are a few new ones though:

- `-d`: run this container as a daemon (aka don't attach to it, just run it). This means that your console won't get blocked when running it.

- `-v /some/content:/usr/share/nginx/html:ro`: this is a bit more interesting. This basically means mounting a folder / file inside the container itself. Let's split the "/some/content:/usr/share/nginx/html:ro" by ":":
    - first part: that is going to be a file / folder on our OS which is going to be mounted inside the container.
    - second part: the destination where that file / folder is going to be mounted. Note that `/usr/share/nginx/html` already exists inside the container in this particular scenario, but we are going to overwrite that with our own.
    - third part: means that the mount will be read-only, meaning that the container itself won't be able to write in that particular folder. This is to avoid potential security issues in which a malicious user would gain access to the container and would try to write files in that folder. You typically want to run things as unprivileged as possible.

Note that containers are supposed to be ephemeral, meaning that they're not expected to exist forever, and thus, their storage is ephemeral as well. This can be an issue in some cases: imagine having a MySQL container, and then imagine it disappearing. We'd lose our databases, which wouldn't be bad. However, the files / folders we mount inside the containers remain even if the containers are deleted. This means that for these cases we'd mount a folder inside the container, and have the application use that folder for storage; the data would be preserved.

But in the `nginx` case, why are we overwriting the `/usr/share/nginx/html` inside the container? That's because we will want to serve our own websites through `nginx`, not the default pages. Let's try it out!

Let's create our own demo page:

```
mkdir mywebsite
nano mywebsite/index.html
```

And add any HTML code:

```
<html>
    <head>
        You expected a proper HTML head, but it was me, DIO!
    </head>
    <body>
        Jonathan's body.
    </body>
</html>
```

Now, we can create an `nginx` container and serve our own HTML page:

```
docker run --name dio-nginx -v "$(pwd)/mywebsite:/usr/share/nginx/html:ro" -d nginx
```

Ok, now our page is being served. But how do we access it? We will first have to get its IP:

```
docker inspect dio-nginx
```

Here we can see details about the container, including its `IPAddress` in the `NetworkSettings` section. Now that we have the IP address, we can access our page (your IP may be different):

```
curl http://172.17.0.3
```

We should be able to see our web page. But, how would our users even know to access that particular IP? And what if the IP changes? Well, there's a better way to actually launch the container: we can map one of the host's port to one of the container's port, which means that whenever the host port's is being accessed, it's actually going to be the container's port. In this case, we won't even need to know the container's IP, just the host's. Containers already have support for this:

```
# stop and remove the previous instance of the container.
docker stop dio-nginx
docker rm dio-nginx

# map host's port 8080 to the container's port 80 (default HTML port).
docker run --name dio-nginx -p 8080:80 -v "$(pwd)/mywebsite:/usr/share/nginx/html:ro" -d nginx

# check that we can access it through our localhost port 8080:
curl http://localhost:8080
```

Great success. We didn't need to find out the container's IP, which is great. Keep in mind that ports can only be used by one application at a time, so while this container is running, we can't assign port `8080` to any other containers. But we can map other ports though, which means we can run multiple instances of nginx containers:

```
docker run --name regular-nginx -p 9090:80 -d nginx

# check that we can access it through our localhost port 9090:
curl http://localhost:9090
```

Great, we're now running multiple containers for various purposes! We're only scratching the surface here though, and there's plenty more to look into. But generally, if you don't know what commands there are in docker, you can always run ``docker --help`` (this also works for pretty much any command too). Additionally, if you don't know how to use a particular command, you can try ``docker command-name --help`` (e.g.: ``docker run --help``).


Building our own container images
---------------------------------

Now, how exactly are these container images structured? How are they created?

Container images are basically just some TAR archives which contains some metadata JSON files and a few other archives or files "overlayed" on top of each other (called "Layers"). We can take a look at such an image:

```
# pull the image and then export the image from docker to a local .tar archive.
docker pull hello-world
docker save -o hello-world.tar hello-world

# extract the archive.
tar -xvf hello-world.tar
ls
```

We should now see the components of a container image. These components respect the OCI (Open Container Initiative) spec, so it's called an OCI image, and it's a standard image. In `manifest.json` we see details about the image's layers, while in `index.json` we see details about the image itself, like its size, type, tags, names, etc:

```
cat manifest.json | jq
cat index.json | jq
```

And then we have the files in `blobs/sha256`. Here are the layers described in `manifest.json`. They can be regular files, json files, other archives, etc.

Now, let's see how we can create our own images.

First of all, we'll have to have an application to package into a container image. Let's create a simple Python application in Flask.

```
mkdir ~/simpleapp
cd ~/simpleapp
nano app.py
```

Add the following Python code:

```
import flask

app = flask.Flask(__name__)


@app.route('/')
def hello():
    return "Hello World! How you doin'?.\n"
```

This code depends on a 3rd party library called `flask`. If we want our application to run, that library will have to be installed. We can write our dependencies in a text file, and have that list of dependencies installed later. Add the `requirements.txt` file and this content in the file:

```
flask
```

Now, we're going to need to create a ``Dockerfile`` file. The name ``Dockerfile`` is standard, so they should be typically be named like so, but there are cases in which different dockerfiles may be needed for differerent OSes. This is typical for cases in which we'd have to build container images for Windows and Linux (they can be quite different).

A Dockerfile is basically a list of instructions on how to build the container image. We typically start with a `FROM` instruction, specifying the base image we're going to build on top of. Imagine a deck of cards; we can choose to build on top of an existing deck of cards, or start from `scratch`. We can build images using `FROM scratch`, as long as we include all the necessary components our application requires. In some instances, this may be preferable due to security concerns (less surface area => fewer chances that there's a vulnerability somewhere), but that can be quite challenging in some cases. We're going to build on top of an existing image. Let's add the following `Dockerfile`:

```
# The base image we're building on top of. This already has python installed.
FROM python:3.10-alpine

# Create folder and cd to that folder.
WORKDIR /code

# Setting some configurations for the flask application.
# Typically, applications may require configuration, and some of them can be configured
# through environment variables.
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0

# Add our files. Also works with URLs.
ADD requirements.txt requirements.txt
ADD app.py app.py

# Run some commands inside the container image.
# In this case, it will install our dependencies.
RUN pip install -r requirements.txt

EXPOSE 5000

# Default command to run.
CMD ["flask", "run", "--debug"]
```

Next, we have to build our image:

```
docker build -t docker.io/yourdockerhubusername/simpleapp:1.0 --platform "linux/amd64" .
```

Note that we're building the image for `linux/amd64`. Additionally, see that the name of the image that we've built is `docker.io/yourdockerhubusername/simpleapp:1.0`. This name has 4 parts in it:

- `docker.io`: This is the registry name, and this refers to DockerHub. This prefix will allow us to push this image to DockerHub if we want. There are other registries as well, but keep in mind that if you want to push to those registries, this prefix has to be changed to match the registry we're pushing to. This part is optional, and if not set, it refers to `docker.io` (the names `docker.io/yourdockerhubusername/simpleapp:1.0` and `yourdockerhubusername/simpleapp:1.0`) are equivalent.
- `yourdockerhubusername`: This is typically for the username / publisher name, and you'll have to set it if you want to push it on your registry in DockerHub or any other registry. Note that some images do not have this part: `hello-world`, `nginx`. That's because those are DockerHub official images, and they've chosen not to have this part for simplicity's sake. But in most scenarios, you'll have to add this part to your container image names.
- `simpleapp`: This is the actual image name.
- `1.0`: This is the image tag. In this case, it's also the image version. Tags typically are set in a way that make sense. For example, in the `python:3.10-alpine` image above, we can deduce that the Python version is `3.10`, and the container image was built on top of an `alpine` container image (though, we cannot deduce from this what version of alpine without further inspection). We might have other tags, like `1.0-linux-amd64`, which could mean that it's the 1.0 image version for `linux/amd64`. If the tag is not explicitly set, it will be automatically be set to `latest`.

Now that we've built the container image, let's run it and test it:

```
docker run --name my-pythony -p 9000:5000 -d yourdockerhubusername/simpleapp:1.0

curl http://localhost:9000
```

Congrats, you've built your first container image! Now, the container image building process can be a lot more complex, depending on what you want to do, and it's a vast topic to cover. We can't get into such details in this training, sadly. But, there's another bit of interesting tech that's useful to know when it comes to container images: manifest lists.

Consider this: it can be quite difficult and headache-inducing to keep track of what images are for what `os/arch` platform. How awesome would it be if you could just have an `you/image:1.0` which could basically work for every single `os/arch` platform combination, and when a user would `docker pull you/image:1.0`, it would get the appropriate image for his platform? Well, that's exactly what manifest lists are for. A manifest list is basically a list of container images grouped into a single name, each image corresponding to a particular `os/platform`. This can be done by running:

```
docker manifest create --amend "gigi/imagename:1.0" "gigi/imagename:1.0-windows-amd64" "gigi/imagename:1.0-linux-amd64"

docker manifest push --purge "gigi/imagename:1.0"
```

The command above would create the the manifest list `gigi/imagename:1.0` which would contain 2 container images, and then it would publish that manifest list in your registry. Keep in mind that the 2 component images should already be published before doing this, otherwise this wouldn't work.

We can inspect other manifest lists published by other people by running:

```
docker run --rm mplatform/mquery claudiubelu/busybox:1.29
```

Output:

```
Image: claudiubelu/busybox:1.29 (digest: sha256:ed264635129fa0c31f3d478761e5426e4d0127ea7a9a2e33ffc5f660b4a3041a)
 * Manifest List: Yes (Image type: application/vnd.docker.distribution.manifest.list.v2+json)
 * Supported platforms:
   - linux/amd64
   - linux/arm/v7
   - linux/arm64
   - linux/ppc64le
   - linux/s390x
   - windows/amd64:10.0.17763.1757
   - windows/amd64:10.0.18362.1256
   - windows/amd64:10.0.18363.1379
   - windows/amd64:10.0.19041.804
   - windows/amd64:10.0.19042.804
```

Here we can see that the image `claudiubelu/busybox:1.29` is a manifest list containing images for different `os/arch` platforms, including multiple Windows images for different versions of Windows (Windows containers are complicated; but in general, the OS Version of the Windows container image should match the OS Version of the Windows host).

There are plenty more to learn about building container images. I had a presentation on building container images (with a focus on Windows container images) at KubeCon: https://www.youtube.com/watch?v=iR6sIf5UOX0


Deploying application requiring multiple containers
---------------------------------------------------

In most scenarios, we will have complex applications built from multiple components. For example, we might have a distributed application in multiple components (API, conductor, workers), which communicate with each other through a messaging protocol like AMQP through an AMQP server like RabbitMQ. This server may use a database (Mysql, MongoDB, Redis, etc.). Each of the component we've mentioned could be in a separate container, and there could be more components than we've already mentioned. This could be quite a lot to deploy manually. Plus, if the host goes down, those containers would have to be restarted manually. Thankfully, there's a nicer way to do this through `docker compose`.

Let's create a new project, in which we'll create a Python web server which requires a Redis database. This basically means there are 2 components to deploy and relate to each other. Let's start:

```
mkdir ~/myapp
cd ~/myapp
```

Let's add the following Python code:

```
import time

import flask
import redis


app = flask.Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    for retry in range(5):
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retry == 4:
                raise exc
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return f'Hello World! I have been seen {count} times.\n'
```

This code has 2 dependencies, `flask`, and `redis`. If we're going to run this application, we're going to have to install them beforehand. So let's write that file:

```
nano requirements.txt
```

And add:

```
flask
redis
```

Next, we're going to need to create the Dockerfile:

```
FROM python:3.10-alpine

WORKDIR /code

ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0

ADD requirements.txt requirements.txt
ADD app.py app.py

RUN apk add --no-cache gcc musl-dev linux-headers
RUN pip install -r requirements.txt

EXPOSE 5000

CMD ["flask", "run", "--debug"]
```

Next, we're going to need to add the `compose.yaml` file:

```
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

And finally, we can deploy everything by running:

```
docker compose up
```

Keep in mind that on the first run, we don't have the container image for our application built, so that will take a bit more time to do, but on subsequent runs, it won't have to rebuild it.

However, if for some reason there were mistakes in our application (e.g.: bugs), the image won't be rebuilt. If you need to force the image rebuild process, run:

```
docker compose up --build
```

We should have the application up and running. In another terminal, you can test it out:

```
curl http://localhost:8000
curl http://localhost:8000
curl http://localhost:8000
curl http://localhost:8000
```

We should be able to see that the hit counter increases. That counter is from the Redis database.


What's next?
------------

There are many more things to learn about containers, we've barely scratched the surface. Plus, we've only looked into how to run containers on a single host. How can we manage multiple hosts, each with their own containers? For that, we could use a Container Orchestrator like Kubernetes, which would be the topic for another time. But for now, know that a lot of what we've learned is being used in cloud computing, and a lot of cloud workloads are currently based on containers. The cloud itself could be another complex topic to cover.
