# docker-compose-laravel
My Docker Compose setup for Laravel projects, inspired by [this repo](https://github.com/aschmelyun/docker-compose-laravel).

## Table of contents
* [docker-compose-laravel](#docker-compose-laravel)
* [Table of contents](#table-of-contents)
* [Setup](#setup)
  * [Prerequisites](#prerequisites)
  * [New Project](#new-project)
  * [Existing Project](#existing-project)
* [Usage](#usage)
* [Configuration](#configuration)
  * [Services](#services)
    * [app](#app)
    * [database](#database)
    * [webserver](#webserver)
  * [Building](#building)
* [Reference](#reference)
  * [Ports](#ports)

## Setup
Setup is fairly straight forward, there are no installation steps for the project itself, just ensure that the prerequisites are met and you are off to the races!

### Prerequisites
Ensure that [Docker is installed](https://docs.docker.com/docker-for-mac/install/) and up to date on your system. Once installed, configure with your required preferences and ensure it is running. Make sure you are logged in to Docker Hub via the app so that it can download the container images.

### New Project
Create a new repo on GitHub, or your git host of choice. Once created, grab and save the remote for later. This will be my example, created the remote and ready to use.

```
New remote: git@github.com:othyn/new-docker-laravel-project.git
```

If it is a new project, clone this repo to a desired directory, making sure that the directory aligns with the new repo name (not essential, but good practice):

```bash
$ git clone git@github.com:othyn/docker-compose-laravel.git ~/git/new-docker-laravel-project
```

Once cloned, time to alter the remote to point at your new project and set the new upstream branch. We will now need that remote you saved to one side earlier:

```bash
$ cd ~/git/new-docker-laravel-project
$ git remote set-url origin git@github.com:othyn/new-docker-laravel-project.git
$ git push -u origin master
```

Awesome, so we now have a new working project directory, you can use the [Laravel installation tool](https://laravel.com/docs/master/installation#installing-laravel) and follow the [Laravel configuration steps](https://laravel.com/docs/master/installation#configuration) to create a new project in the `src` directory:

```bash
$ cd ~/git/new-docker-laravel-project
$ laravel new src --force
```

Done! Now you can get to building that Laravel app you've always wanted to.

### Existing Project
For existing projects its a little more complex. At this point in time, just to keep the project structure the same, I would recommend cloning the repo and then manually copying across the files into their new locations. So, this will mean moving the cloned files across into the old repo, then moving the Laravel project into `src`.

If you have any better suggestions, please feel free to submit them! Such as making this repo into a git submodule.

## Usage
Once the project has been [Setup](#setup), it's very simple to use. Lauch docker composer from within the root directory of the project, the one with the `docker-compose.yml` file in it and away you go! The first time it is run, docker will build the containers, so it may take a little while longer.

```bash
$ docker-compose up
```

If you wish to run them in the background, in `detached` mode so that they don't remain as open processes within your terminal window, add the `-d` flag to the end of the command `$ docker-compose up -d`.

If you wish to detach from an already running `$ docker-compose up`, use `CTRL` + `Z`, that will suspend the process into the background. When in the background it becomes part of the `jobs` queue. You can view them and their `[id]` using the following command:

```bash
$ jobs
> [1]  + suspended  docker-compose up
#  ^ That is the ID of the process.
```

Then to get back to a background job, use its ID prefixed with a percent sign in the following command, it will return the process to the foreground:

```bash
$ fg %1
#     ^ That is the ID of the process.
```

To instead kill a background task, run the following, using that process ID that we had from before:

```bash
$ kill -KILL %1
#             ^ That is the ID of the process.
```

To re-attach to the docker logs whilst the process is suspended or `detached` (the output that `$ docker-compose up` would usually sit you at), run the following:

```bash
$ docker-compose logs -f -t
```

That will attach you back to the logs. To do this for a specific container, add the container name to the end of the command:

```bash
$ docker-compose logs -f -t <container>
# container can be any running container. e.g. webserver, database or app
```

To run a command in a running the container:

```bash
$ docker-compose exec <container> <command>
# container can be any running container. e.g. webserver, database or app
# command can be any command. e.g. 'top' or 'sh' (Alpine) / 'bash' (Ubuntu) to enter an interactive shell
```

To stop the containers, if they are attached simply press `CTRL` + `C` which is the escape sequence for any CLI application. That should gracefully stop them, if it aborts or the containers are running in `detached` mode, do the following:

```bash
$ docker-compose down
```

You can use `stop` instead of `down` to just stop the running container. The above command, `down`, will both stop and remove the container and its associated networks. You can also specify `--volumes` as an additional flag to remove any associated volumes, and the `--rmi <all|local>` flag to remove associated images.

## Configuration
There are elements of this docker project that you can configure for you project if you require extra functionality. Obviously, at the end of the day this is just a bog standard Docker project, so you can go to town with any changes you wish. But these are the main areas that are easy adaptable.

### Building
Should you change parts of the docker container, make sure to re-build the containers!

```bash
$ docker-compose build
```

### Services
This is the configuration for all of the core services that are configured, the `app`, `database` and `webserver`. All the services configuration is, as usual, located in their declaration within the `docker-compose.yml` in the project root, and by a directory with the service name in the `docker` directory in the root directory.

#### app
The `dockerfile` for the `app` contains all provisioning steps within the build process for the container, so add anything you wish to be built into it in there, as per the Docker docs.

The `entrypoint.sh` script is copied in and executed when the container is brought `up`, so this runs things like `artisan` commands (migrations, seeders, etc.) and such to get Laravel in a ready state. Add anything in to this file that you need running every time the container is upped, not built.

The `php.ini` file is any PHP ini configuration you wish to set, this overwrites the system default `php.ini`.

#### database
The `persist` directory is volume mapped to the MySQL directory on the docker container, so that the containers DB is persisted across container instances.

The `base.sql` patch file is run by the MySQL Docker container when its upped, so place any SQL statements in there that you wish to be run. E.g. creating databases.

The `mysql.conf` file is any MySQL configuration you wish to set, this overwrites the system default `mysql.conf`.

#### webserver
The `nginx.conf` file is any NGINX configuration you wish to set, this overwrites the system default `nginx.conf`.

## Reference
This will just have things of reference for the project and a brief explanation on why things exist should it be necessary.

### Ports
Here are the exposed port maps for the containers:

| Service   | Host Port | Container Port |
|-----------|-----------|----------------|
| webserver | 8080      | 80             |
| database  | 3306      | 3306           |
| app       | 9000      | 9000           |
