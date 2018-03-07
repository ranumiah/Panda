docker-compose build	==> create a new container
docker images			==> show all the images on the machine
docker-compose up		==> putting the container up
docker-compose down		==> taking the container down
docker-compose ps -a	==> showing all the docker running

One liner to stop / remove all of Docker containers:
	docker stop $(docker ps -a -q)
	docker rm $(docker ps -a -q)


How is Docker different from a normal virtual machine?

Docker originally used LinuX Containers (LXC), but later switched to runC (formerly known as libcontainer), which runs in the same operating system as its host. This allows it to share a lot of the host operating system resources. Also, it uses a layered filesystem (AuFS) and manages networking.
AuFS is a layered file system, so you can have a read only part and a write part which are merged together. One could have the common parts of the operating system as read only (and shared amongst all of your containers) and then give each container its own mount for writing.
So, let's say you have a 1 GB container image; if you wanted to use a full VM, you would need to have 1 GB times x number of VMs you want. With Docker and AuFS you can share the bulk of the 1 GB between all the containers and if you have 1000 containers you still might only have a little over 1 GB of space for the containers OS (assuming they are all running the same OS image).
A full virtualized system gets its own set of resources allocated to it, and does minimal sharing. You get more isolation, but it is much heavier (requires more resources). With Docker you get less isolation, but the containers are lightweight (require fewer resources). So you could easily run thousands of containers on a host, and it won't even blink. Try doing that with Xen, and unless you have a really big host, I don't think it is possible.
A full virtualized system usually takes minutes to start, whereas Docker/LXC/runC containers take seconds, and often even less than a second.
There are pros and cons for each type of virtualized system. If you want full isolation with guaranteed resources, a full VM is the way to go. If you just want to isolate processes from each other and want to run a ton of them on a reasonably sized host, then Docker/LXC/runC seems to be the way to go.
For more information, check out this set of blog posts which do a good job of explaining how LXC works.

Why is deploying software to a docker image (if that's the right term) easier than simply deploying to a consistent production environment?

Deploying a consistent production environment is easier said than done. Even if you use tools like Chef and Puppet, there are always OS updates and other things that change between hosts and environments.
Docker gives you the ability to snapshot the OS into a shared image, and makes it easy to deploy on other Docker hosts. Locally, dev, qa, prod, etc.: all the same image. Sure you can do this with other tools, but not nearly as easily or fast.
This is great for testing; let's say you have thousands of tests that need to connect to a database, and each test needs a pristine copy of the database and will make changes to the data. The classic approach to this is to reset the database after every test either with custom code or with tools like Flyway - this can be very time-consuming and means that tests must be run serially. However, with Docker you could create an image of your database and run up one instance per test, and then run all the tests in parallel since you know they will all be running against the same snapshot of the database. Since the tests are running in parallel and in Docker containers they could run all on the same box at the same time and should finish much faster. Try doing that with a full VM.
From comments...
Interesting! I suppose I'm still confused by the notion of "snapshot[ting] the OS". How does one do that without, well, making an image of the OS?

Well, let's see if I can explain. You start with a base image, and then make your changes, and commit those changes using docker, and it creates an image. This image contains only the differences from the base. When you want to run your image, you also need the base, and it layers your image on top of the base using a layered file system: as mentioned above, Docker uses AUFS. AUFS merges the different layers together and you get what you want; you just need to run it. You can keep adding more and more images (layers) and it will continue to only save the diffs. Since Docker typically builds on top of ready-made images from a registry, you rarely have to "snapshot" the whole OS yourself.


Images are foundation of a containers…
Docker-Daemon

Docker pull <<Image_Name>> will get an image, which are really small…
Docker images  list all images
Docker run <<Image_Name>> will run the image in a container
Docker ps  list all the container that is running.
Docker ps –a  all the container on the machine regardless or it is running or not.
Docker rm <<Container_ID>>  remove a container
Docker rmi <<Image_ID>>  remove an image


An Image is a set of read-only layers, whereas a container has a thin read/write layer.

A volumn is a special type of directory in a container typically referred to as a "data volume".  It can be shared and reused among containers and any updates to an image won't affect a data volumne.
Also, data volumes are persistent. So even if a container is deleted and it's completely blow away from the machine, the data volume can still stick around and you have control over that.


docker run -p 8080:3000 -v C:\temp\docker $(pwd):<<Image_Name>>
$(pwd):	=> current working directory

# Delete All Machines
docker-machine rm -y $(docker-machine ls -q)

# Create default hypver-v machine
docker-machine create -d hyperv --hyperv-virtual-switch "<NameOfVirtualSwitch>" <nameOfNode>

docker-machine create --driver hyperv default --hyperv-virtual-switch "To Internet"

# Delete every Docker containers
# Must be run first because images are attached to containers
docker rm -f $(docker ps -a -q)

# Delete every Docker image
docker rmi -f $(docker images -q)

### Docker Client

Some useful command regarding the client

`docker help` -- This will give you all the available commands. It provides a quick description of what each commands does.

`docker <COMMAND> --help` This provide more information or how to utilise a particular command.

`docker version` -- This will show what version of Docker installed on your machine as well as if your running a linux or windows container.

`docker info` -- This has lot more useful information like number of container and its state as well as the number of images.

### Docker Images

`docker images` -- List all the images locally downloaded.

`docker pull <IMAGE NAME>` -- If you do not have the image on your local machine then it will download the image from Docker hub. However it will not start a container.

`docker search <SEARCH TERM>` -- This is equivalent to vising [Docker Hub](https://hub.docker.com/).

### Docker Container

* `docker create <IMAGE NAME>` -- Create a container from an image without starting it.
* `docker start <CONTAINER ID>` -- Start up a container.
* `docker run <CONTAINER ID>` -- Creates and starts a container.
* `docker stop <CONTAINER ID>` -- Stop a container.
* `docker rename <CONTAINER NAME> <NEW NAME>` -- Change the name of the container.
* `docker attach <CONTAINER ID>` -- This will connect to a running container.
* `docker ps` -- This gives you a list of all the running containers.
* `docker ps -a` -- This will list all running and exited containers.

### Docker Run

If you wish to download and run then the following command will be more appropriate. `docker run --rm -it microsoft/dotnet:2.1-sdk dotnet --version`

The arguments are as follows:

* `--rm` -- It will delete the container and not the image when we exit the container.
* `-it` -- create an "interactive" session and attaches the tty input.
* `microsoft/dotnet` -- the name of the image.
* `2.1-sdk` -- This is the tag, which means what version of image to use.
* `dotnet --version` -- The command to execute in the container when it starts.

Other useful parameter you can use with `docker run`:

* `-d` -- Running the container in Detached mode.
* `--name <CONTAINER NAME>` -- This will give a name to the container instead of the default image name if omitted.
* `-v <HOST DIR>:<CONTAINER DIR>` -- Map the Host and Container folder.
*`-p <HOST PORT>:<CONTAINER PORT>` -- This is port forwarding from between Host and Container.

As of writing this blog on windows you can not use localhost:8080 but instead you will need to use the container ip address. This can be found using: `docker inspect <CONTAINER ID>`

Running the following command will change between Linux and Windows containers

    & 'C:\Program Files\Docker\Docker\DockerCli.exe' -SwitchDaemon

### Housekeeping

`docker system prune` -- This will remove all *unused* and *dangling* images, containers and build cache.

`docker rm -f $(docker ps -a -q)` This will delete all Docker containers.

`docker rmi -f $(docker images -q)`This will delete all Docker images.

docker version
docker info

mklink /j C:\ProgramData\Docker D:\Docker
open C:\ProgramData\Docker\config\daemon.json and add
    {
    "graph": "D:\\ProgramData\\Docker"
    }
This will put all your images, layers and containers in a different location but leave the config in its original location

This will run the container interactively and once finished it will remove it.  If the image exist it use it otherwise it will pull it down. Finally before finishing it will execute in command prompt "dotnet"
    docker run --rm -it microsoft/dotnet:2-runtime dotnet
If you don't provide a command i.e. 'dotnet' then it will default to something in this case a command prompt




    docker run --rm -it -v ${PWD}:C:\api microsoft/dotnet:2-runtime

-v ${PWD}:C:\api    ==> This will take the current working directory (PWD in powershell) from the host and MAP it to the Container (C:\api)
Therefore left represent the HOST & right represent the CONTAINER


git clone this: https://github.com/g0t4/aspnetcore-generator-api
cd to api folder and run ==> dotnet publish
docker run --rm -it -v ${PWD}:C:\api microsoft/dotnet:2
cd to api-publish folder in the containter and type the following:
    dotnet .. api.dll
    This will run the webapi application in the container

On host machine run the following
    docker ps   ==> get the container id
    docker inspect <above container id>
Grap the Container IP Address:
    NetworkSettings ==> Networks ==> nat ==> IPAddress

docker run --rm -it -v ${PWD}:C:\api -p 8080:80 microsoft/dotnet:2
-p ==> This is port fowrading which will expose 8080 on host and port 80 in the container. This only works on Linus Container. However on windows 10 Creator Update or above you cannot use localhost:8080. Instead you will need to use ipconfig and get the ip like: 192.168.1.64:8080

docker run --rm -it -v ${PWD}:C:\api -p 8080:80 microsoft/dotnet:2

docker run --rm -it -v ${PWD}:C:\api -p 8080:80 microsoft/aspnetcore-build:2
    cd /api
    rm -rf bin/ obj/
    dotnet restore
    dotnet publish
run -v ${PWD}:C:\api -p microsoft/aspnetcore-build:2

fsutil file createnew Dockerfile 0
fsutil file createnew .dockerignore 0

.dockerignore ==> ignore everything except the publish folder
    *
    !bin/Debug/netcoreapp2.0/publish

How to build Docker File?
    docker build -t aspnetcore/generator:build .
How to run the image now?
    docker run --rm -it -p 8080:80 aspnetcore/generator:build
by default docker uses Caching...


Build-Environment
Runtime-Environment


Docker compose by default uses a docker-compose.yml file.
    docker-compose up ==> will create and attach the container
    docker-compose down ==> remove the container
    docker-compose -d ==> same as above but in detach mode
    docker-compose logs ==> shows the logs
    docker-compose logs -f ==> this always to follow the logs
    docker-compose up --build ==> will build the container everytime and not only for the first run

docker-compose up -d --build


$command = Get-History -Count 1
$command.EndExecutionTime - $command.StartExecutionTime

This will keep the the container alive even when you close Visual Studio.
    tail -f /dev/null