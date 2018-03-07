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

    With your text editor create a new file called `Dockerfile` in the same directory as your `index.html`

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
    Yarn allows us to run packages directly and not have to install them globally or mess around with our path env variable. This is neat as it allows us to list all, normally globally installed, dependencies within our package.json alonside our other dependencies. <br>

    For the above example we could
    - add the package globally
    - install it as we did then add a script in the package.json
    - we could run the command "node node_modules/.bin/serve"...

    Alternatively and much simpler just "yarn serve"

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

    > **Don't close your running Docker Image** <br>
    We need it for the next exercise

1. Data in containers is not persisted

    Objectives:
    - Realize data held in a cotainer is not persisted

    In the same node:alpine image from the previous step

    ```sh
    # Change up a directory so you are in /usr/app/
    /usr/app/cabbage-savage $# cd ..

    # Make a new directory
    /usr/app $# mkdir this_will_not_persist

    # Check the dirs available
    /usr/app $# ls -l
    total 4
    drwxr-xr-x    6 root     root           204 Mar  5 00:01 cabbage-savage
    drwxr-xr-x    2 root     root          4096 Mar  6 00:02 this_will_not_persist
    ```

    Note how we have two directories in the /usr/app directory - `cabbage-savage` and `this_will_not_persist`

    ```sh
    # Close the container
    ctrl + d

    # Restart the container as before
    $ docker run -p 1234:5000 -v $(pwd):/usr/app/cabbage-savage -it node:alpine sh

    # change directory
    / $# cd /usr/app/

    # Check the dirs available
    /usr/app $# ls -l
    total 4
    drwxr-xr-x    6 root     root           204 Mar  5 00:01 cabbage-savage
    ```

    Note how the directory `this_will_not_persist` disappeared.

    > **Pets vs Cattle** http://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/ <br>
    "In the old way of doing things, we treat our servers like pets, for example Bob the mail server. If Bob goes down, it’s all hands on deck. The CEO can’t get his email and it’s the end of the world. In the new way, servers are numbered, like cattle in a herd.<br>
    For example, www001 to www100. When one server goes down, it’s taken out back, shot, and replaced on the line."

    Data in containers is not persisted. This is handy as our containers can be stopped and started and always start in a predictable state.

1. Using an attached container to start a project

    Objectives:
    - Realize the short-comings of a Dockerfile-first methodology.

    Initialize the project and install all prequired packages via the attached docker container.

    Advantages:
    - Some node packages contist of binaries that are build for specific OS's.
    - If some of the team is on a slightly different version of node, this can effect the versions of the packages that are installed.
    - As a result, your package-lock.json or yarn.lock may vary from environment to environment.
    - By running the install via the container, you're ensuring the packages are being installed for the correct environment.
    - All users run their app - be it for dev or prod - via the same container.
    - Eliminates the need for version managing tools live rvm (ruby), nvm (node), pyenv (python), etc

    ```sh
    # Change to the cabbage-savage directory
    $ cd ~/docker-prac/cabbage-savage

    # Remove all files in the cabbage-savage directory, EXCEPT the index.html
    $ [[ "$PWD" == *cabbage-savage ]] && find . \! -name "index.html" -delete

    # Start a Node Alpine container (we don't require ports right now)
    $ docker run -v $(pwd):/usr/app/cabbage-savage -it node:alpine sh

    # Go to the working directory
    / $# cd /usr/app/cabbage-savage

    # Initialize a new node project - hit return for all the fields
    /usr/app/cabbage-savage $# yarn init
        yarn init v1.5.1
        question name (cabbage-savage):
        question version (1.0.0):
        question description:
        question entry point (index.js):
        question repository url:
        question author:
        question license (MIT):
        question private:
        success Saved package.json
        Done in 5.47s.
    ```

    Note at this point, your mapped volume will now contain a `package.json` file.

    ```sh
    # Install serve
    /usr/app/cabbage-savage $# yarn add serve
        yarn add v1.5.1
        info No lockfile found.
        [1/4] Resolving packages...
        [2/4] Fetching packages...
        [3/4] Linking dependencies...
        [4/4] Building fresh packages...
        success Saved lockfile.
        success Saved 134 new dependencies.
        ...
        Done in 8.98s.

    # Exit the container
    ctrl + d
    ```

    Your mapped volume will now also contain a `yarn.lock` file and `node_modules` directory.

    You've setup the project and are now ready to package it all into `cabbage-savage-image`!

1. Our second Dockerfile

    Objectives:
    - Realize there are layers
    - Optimize image build size

    With your text editor create a new file called `Dockerfile` in the directory `cabbage-savage` directory

    ```yaml
    # Specify the image
    FROM node:alpine

    # Specify the working directory
    WORKDIR /usr/app/cabbage-savage/

    # Copy the package.json and *.lock files to the working directory
    COPY package.json *.lock ./

    # Run the yarn install
    RUN yarn install

    # Copy the index.html file to the working directory (or all working files)
    COPY index.html ./

    # Run a start command
    CMD yarn serve

    ```

    Build and tag your image

    ```sh
    # Assuming you are still in ~/docker-prac/cabbage-savage
    $ docker build -t cabbage-savage-image .
        Sending build context to Docker daemon  12.14MB
        Step 1/6 : FROM node:alpine
        ---> 2f9669a41a9f
        Step 2/6 : WORKDIR /usr/app/cabbage-savage/
        ---> Using cache
        ---> c7b27d12f574
        Step 3/6 : COPY package.json *.lock ./
        ---> Using cache
        ---> 5588051178a8
        Step 4/6 : RUN yarn install
        ---> Using cache
        ---> 1b590ed33812
        Step 5/6 : COPY index.html ./
        ---> Using cache
        ---> 767f88769dc5
        Step 6/6 : CMD yarn serve
        ---> Using cache
        ---> 3b2b1e2e4cb9
        Successfully built 3b2b1e2e4cb9
        Successfully tagged cabbage-savage-image:latest
    ```

    > **Docker Layers** https://medium.com/@jessgreb01/digging-into-docker-layers-c22f948ed612 <br>
    "Layers of a Docker image are essentially just files generated from running some command.
    Layers are neat because they can be re-used by multiple images saving disk space and reducing time to build images while maintaining their integrity"

    Each one of the steps above produced a sequencial layer. If any one of the files or layers change, the proceeding layers will be rebuilt. This is great because if you look at the structure of the Dockerfile, we wouldn't need to install node packages everytime because they don't change that often. Consequently we are likely to change the working files and would want that really close the end of the build process so it doesnt force the app to re-download all the node modules everytime.

    Let's check the size of the image:

    ```sh
    # Check all your images
    $ docker images
    ```

    | REPOSITORY| TAG| IMAGE ID| CREATED| SIZE |
    | --- | --- | --- | --- | ---  |
    | cabbage-savage-image | latest | 3b2b1e2e4cb9 | 2 minutes days ago | 92.6MB |
    | cool-doggo-image | latest | f660a3442d6e | 2 minutes days ago | 17.9MB |
    | nginx | alpine | 537527661905 | 12 days ago | 17.9MB |
    | nginx | latest | e548f1a579cf | 12 days ago | 109MB |

    `cabbage-savage-image` - 92.6MB!

    Let's see if we can reduce that size:

    ```yaml
    # Specify the image
    FROM node:alpine

    # Specify the working directory
    WORKDIR /usr/app/cabbage-savage/

    # Copy the package.json and *.lock files to the working directory
    COPY package.json *.lock ./

    # Run the yarn install
    RUN yarn install

    # Minimize size
    RUN apk del --purge --force libc-utils \
    && rm -rf /var/lib/apt/lists/* \
        /var/cache/apk/* \
        /usr/share/man \
        /tmp/* \
        /usr/lib/node_modules/npm/man \
        /usr/lib/node_modules/npm/doc \
        /usr/lib/node_modules/npm/html \
        /usr/lib/node_modules/npm/scripts \
        $(yarn cache dir)

    # Copy the index.html file to the working directory (or all working files)
    COPY index.html ./

    # Run a start command
    CMD yarn serve

    ```

    | REPOSITORY| TAG| IMAGE ID| CREATED| SIZE |
    | --- | --- | --- | --- | ---  |
    | cabbage-savage-image | latest | 46dd5d7302d0 | 2 seconds ago | 92.6MB |
    | < none > | < none > | 3b2b1e2e4cb9 | 2 minutes ago | 92.6MB |
    | cool-doggo-image | latest | f660a3442d6e | 2 minutes ago | 17.9MB |
    | nginx | alpine | 537527661905 | 12 days ago | 17.9MB |
    | nginx | latest | e548f1a579cf | 12 days ago | 109MB |

    `cabbage-savage-image` still 92.6MB?

    That's because it didn't reduce the size of the layer pervious to the minimize step - removing files after they've been added doesn't reduce the file size.

    Ammend your file as below

    ```yaml
    ...

    # Run the yarn install
    # and Minimize size
    RUN yarn install \
    && apk del --purge --force libc-utils \
    && rm -rf /var/lib/apt/lists/* \
        /var/cache/apk/* \

    ...

    ```

    Build and tag your image as before, then look at your images:

    | REPOSITORY| TAG| IMAGE ID| CREATED| SIZE |
    | --- | --- | --- | --- | ---  |
    | cabbage-savage-image | latest | 5a55a4441eae | 2 seconds ago | 78.6MB |
    | < none > | < none > | 3b2b1e2e4cb9 | 2 minutes ago | 92.6MB |

    We managed to reduce the file size by 20mb just by clearing out the node cache and other unnecessary files.

    To find out what files to remove, I suggest looking at documentation for the respective language you're working with, looking at other dockerfiles, and searching through the container OS docs.

    Again smaller containers deploy quicker.

    ```sh
    # You can run cabbage-savage-image like before:
    $ docker run -p 1234:5000 cabbage-savage-image
    ```

1. Docker Compose file

    Objectives:
    - Reverse-proxy requests from Nginx to Live-server Container

    In

1. Spin up a docker container with Django
    - Distribute the stock app


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
