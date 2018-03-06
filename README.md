# Docker Practical

<!-- Introduction -->

1. Spin up a docker container

    **Objectives:**
    - What is Nginx
    - What is Dockerhub
    - How to choose a docker image
    - Comparing image sizes

    Start Docker

    ```sh**
    # Is docker running?
    #    If nothing is displayed you may either need to start or install docker
    $ docker -v
        Docker version 17.12.0-ce, build c97c6d6
    ```

    > **What is Nginx?** https://www.nginx.com/resources/glossary/nginx/ <br>
    Nginx is a super lightweight and fast webserver that requires little configuration.

    > **Why Nginx?** <br>
    It requires little configuration and you'll see later that when we have multiple containers, it's easy to setup a reverse-proxy. <br>
    If unsure of it - don't worry. Think of it as a black-box where we plug in files one side and it distributes them on a port the other side - like a Node Express, or a Python Django server would.

    Get the [Nginx image from dockerhub](https://hub.docker.com/_/nginx/)

    > **Dockerhub** https://hub.docker.com/ <br>
    Dockerhub is an online registry of docker images. Dockerhub contains both official and user created images. For the purpose of this tutorial we will only be looking at official images.

    > **Which Image?** https://docs.docker.com/docker-hub/official_repos/#should-i-use-official-repositories <br>
    Personally I prefer working from Alpine Linux images as they are smaller and usually quicker to run in a pipeline. There are other varieties and most standard images are Debian based.

    ```sh
    # Get the Nginx Image
    #   docker run image_name[:tag]
    #   if no :tag is specified docker will take the latest available.
    $ docker pull nginx
        Using default tag:  # <== Note it defaulted
        latest: Pulling from library/nginx
        .
        .
        .
        Status: Downloaded newer image for nginx:latest
    ```

    ```sh
    # Get the Alpine Nginx Image
    $ docker pull nginx:alpine
        alpine: Pulling from library/nginx
        .
        .
        .
        Status: Downloaded newer image for nginx:alpine
    ```

    ```sh
    # Lets look at the size difference
    $ docker images
    ```

    | REPOSITORY| TAG| IMAGE ID| CREATED| SIZE |
    | --- | --- | --- | --- | ---  |
    | nginx | alpine | 537527661905 | 12 days ago | 17.9MB |
    | nginx | latest | e548f1a579cf | 12 days ago | 109MB |

    Note how the alpine Linux image is alost 1/5th of the size. The size is important if building multiple pipelines (without caching), or have limited memory on your system it can make a significant difference in the speed of getting your container up and running.

    Note how the image is made of multiple layers. The layers are the various commands used to create the image. We will cover this more in depth later.


1. Get the Nginx Welcome

    **Objectives:**
    - Learn Port mapping

    ```sh
    # Run the alpine image
    $ docker run nginx:alpine

    # Now in a separate terminal - see what containers are running
    $ docker ps
    ```

    | CONTAINER ID | IMAGE | COMMAND | CREATED | STATUS | PORTS | NAMES |
    | --- | --- | --- | --- | --- | --- | --- |
    | fd... | nginx:alpine | "nginx ..." | now | x seconds | 80/tcp | xxx_xxx |

    Note that the first terminal is blank, but we are running a docker process. The first terminal is actually the output of the nginx container. It's also using port 80 - lets try opening it.

    Try opening <http://localhost:80>

    You should not be able to see anything. This is because by default Docker containers can make connections to the outside world, but the outside world cannot connect to containers.

    If you want containers to accept incoming connections, you will need to provide special options when invoking docker run. Let's try mapping the port via the command line.

    ```sh
    # Close running instance
    ctrl + c

    # Run the alpine image with port mapping (-p host_port:container_port)
    $ docker run -p 80:80 nginx:alpine
    ```

    You'll see that we can now open <http://localhost:80> and see the Nginx Welcome

    Port mapping is a pretty powerful tool. We can map the specific port of the container to anything of the host

    ```sh
    $ docker run -p 1234:80 nginx:alpine
    ```

    Maps port <http://localhost:1234> -- the host machine port 1234 -- to the images port 80

    > **Why port 80?** https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol<br>
    The HTTP protocol normally uses port number 80. Hence webservers typically listen on port 80. Some OS's require high priveledges to connect to port 80 and this is one of the reasons framework webservers such as Express and Django default to something other than port 80.

1. Change the Nginx Welcome

    **Objectives:**
    - Volume mapping

    Similar to port mapping, we can also map data and files from our host computer. We can also create volumes to be shared amongst multiple containers.

    > Note that you can **ONLY** map directories - not individual files.

    The flag and structure for mapping a volume is similar to that of mapping ports:

    ```sh
    ... -v host_location:image_location ...
    ```

    The location of the Nginx welcome screen as per documentation on dockerhub is `/usr/share/nginx/html`. Lets create a small `index.html` and map is across.


    ```sh
    # Create a new directory called cool-doggo
    $ mkdir cool-doggo

    # Now open with the text editor of your choice
    $ [code | idea | vim | atom] cool-doggo
    ```

    Create a new file called `index.html` and paste the following:

    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>Cool Doggo</title>
      </head>
      <body>
        <img src="https://media.giphy.com/media/mRB9PmJFOjAw8/giphy.gif">
      </body>
    </html>

    ```

    Save it and head back to your editor.

    Now lets run docker with the volume mapping.

    ```sh
    # Note that docker requires the full path of your volume thats going to be mapped.
    #   For this project I've created a directory called docker-prac in my home directory
    #   to access cool-doggo I'd look in:
    #   ~/docker-prac/cool-doggo OR
    #   $(pwd) if I cd'd into the "docker-prac/cool-doggo" directory
    docker run -p 1234:80 -v ~/docker-prac/cool-doggo:/usr/share/nginx/html nginx:alpine
    ```

    Open <http://localhost:1234> to view cool doggo!

1. Create a dockerfile and image

    Objectives:
    - Create a dockerfile
    - Tag an image

    With your text editor create a new file called `Dockerfile`

    ```yaml
    # Specify the image
    FROM nginx:alpine

    # Copy the index.html file
    COPY index.html /usr/share/nginx/html
    ```

    Build and tag your image

    ```sh
    # Assuming you are in ~/docker-prac
    $ docker build -t cool-doggo-image ./cool-doggo

        Sending build context to Docker daemon  3.072kB
        Step 1/2 : FROM nginx:alpine
          ---> 537527661905
        Step 2/2 : COPY index.html /usr/share/nginx/html
          ---> f660a3442d6e
        Successfully built f660a3442d6e
        Successfully tagged cool-doggo-image:latest
    ```

    What happened there?

    | Command | Explanation |
    | --- | --- |
    | `docker build` | runs the build command |
    | `-t cool-doggo-image` | is tagging the build result "cool-doggo-image" |
    | `./cool-doggo` | is the context of the build. This assumes there is a "Dockerfile" in the directory. All copy comands in the docker file will be run in context to this location |

    ```sh
    # Check all your images
    $ docker images
    ```

    | REPOSITORY| TAG| IMAGE ID| CREATED| SIZE |
    | --- | --- | --- | --- | ---  |
    | cool-doggo-image | latest | f660a3442d6e | 2 minutes days ago | 17.9MB |
    | nginx | alpine | 537527661905 | 12 days ago | 17.9MB |
    | nginx | latest | e548f1a579cf | 12 days ago | 109MB |

    Note how we have a new "cool-doggo-image"!

    You can run it like before:

    ```sh
    $ docker run -p 1234:80 cool-doggo-image
    ```

    Note how we did not have to map an volumes. This is now a fully packaged app. All we have to do map a port and run it and it will do exactly what we expect on any environment.

    We could map the volume like we did before but the `COPY index.html /usr/share/nginx/html` step copied the index.html into our image.

    Mapping volumes are great for testing live changes, but ultimately you would want to lock it down -- package the application and deploy it to various testing stages and finally production.

    > **The 12 Factor App** https://12factor.net/<br>
    "Minimize divergence between development and production, enabling continuous deployment for maximum agility"

    > **As a side exercise:**<br>
    Try deploying this container to <https://sloppy.io> using [dockerhub](https://hub.docker.com) (you may also require <http://xip.io/>)


1. Spin up a docker container with Node

    Objectives:
    - Distribute a hello world using a node server

    So if we wanted to create a new and more complex image, we could do it straight with a new Dockerfile. I prefer to do it via the commandline first. Build the various layers can take a while and any errors can make the process frustrating and slow.

    Like before in our docker-prac directory...

    ```sh
        # Create a new directory called cabbage-savage
    $ mkdir cabbage-savage

    # Now open with the text editor of your choice
    $ [code | idea | vim | atom] cabbage-savage
    ```

    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>Cabbage Savage</title>
      </head>
      <body>
        <img src="https://media.giphy.com/media/WLbtNNR5TKJBS/giphy.gif">
      </body>
    </html>

    ````

    Lets start a Node Alpine image

    ```sh
    $ docker run node:alpine
      Unable to find image 'node:alpine' locally
      alpine: Pulling from library/node
      605ce1bd3f31: Pull complete
      8831d46bdfbd: Pull complete
      c97b98164030: Pull complete
      Digest: sha256:c6c2bbd87a7e142b5d991a2e860ed808756ed67a7e63a2281e8fbc246b5aed00
      Status: Downloaded newer image for node:alpine
    ```

    And like before -

    ```sh
    # Now in a separate terminal - see what containers are running
    $ docker ps
    ```

    This time the node image exited immediately. This is because node didnt have a process to run. We can specify one in numerous ways.

    ```sh
    $ docker run -it node:alpine node

    # Gives us --
    >
    ```

    This is the node REPL in the node:alpine container

    What happened there?

    | Command | Explanation |
    | --- | --- |
    | `docker run` | runs the docker image |
    | `-it` | interactive mode |
    | `node:alpine` | node alpine image |
    | `node` | start by running the node command |

    This is neat as we could run a shell command, experiment with the container and then write a working Dockerfile.

    Lets try install `npm serve`

    > **Serve (npm package)** https://www.npmjs.com/package/serve <br>
    Neat little node static site server. Can also use express or live-server.

    ```sh
    # Get into the right directory
    cd ~/docker-prac/cabbage-savage

    # Run the docker command:
    #   interactive
    #   port mapped 1234:5000 (5000 as per npm serve documentation)
    #   volume to /usr/app - docker will create this if it does not exist
    #   sh - as we want to run npm installs and scripts
    $ docker run -p 1234:5000 -v $(pwd):/usr/app/cabbage-savage -it node:alpine sh

    # terminal will respond with - we're talking to the node:alipne image shell!
    / $#
    ```

    ```sh
    # change to the cabbage-savage directory
    / $# cd /usr/app/cabbage-savage

    # install serve with yarn
    /usr/app/cabbage-savage $# yarn add serve
    ```

    > **Why Yarn?** <br>
    Yarn allows us to run packages directly and not have to install them globally. This is neat as it allows us to list all dependencies within our package.json and not have to add our node_modules to or env path. <br>
    For the above example we could add the package globally, install it as we did then add a script in the package.json to then run, or we could run the command "node node_modules/.bin/serve"... <br>
    Alternatively just "yarn serve"

    Note this process creates a new node_modules folder inside `~/docker-prac/cabbage-savage` (on your local machine) - this is because of the volume mapping.

    ```sh
    /usr/app/cabbage-savage $# yarn serve

        yarn run v1.5.1
        warning package.json: No license field
        $ /usr/app/cabbage-savage/node_modules/.bin/serve

          ┌────────────────────────────────────────────────┐
          │                                                │
          │   Serving!                                     │
          │                                                │
          │   - Local:            http://localhost:5000    │
          │   - On Your Network:  http://172.17.0.2:5000   │
          │                                                │
          └────────────────────────────────────────────────┘
    ```

    Visit <http://localhost:1234>




1. Create a dockerfile #2

    Objectives:
    - Realize there are layers
    - Optimize image build size

1. Spin up a docker container with Django
    - Distribute the stock app
1. Docker Compose file
    - Reverse-proxy requests from Nginx to Live-server Container


---

## Cheat Sheet


### Docker

docker run -p host_port:image_port image_name[:tag]

See all containers that are running
`docker ps`

Run image with port mapping (-p host_port:container_port)
docker run -p host_port:container_port image_name:tag

??Stop all docker Containers and Images

`docker rm $(docker ps -a -q)`

?? Remove all Images

`docker rmi $(docker images -q)`

?? Remove all Images with force

`docker rmi -f $(docker images -q)`
