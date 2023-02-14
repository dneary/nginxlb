# Testing nginx load balancing with docker-compose

This project is a very basic configuration to run one nginx instance as a proxy, load balancing across
3 other nginx instances serving a static HTML file.
 
1.  `proxy`: We map port 80 of `localhost` to port 80 of the `nginx` container. This container has a
    minimal configuration, defining an `upstream` host group called `web` consisting of three nodes
    `web[123]` and proxying all traffic to them.
2.  `web[123]`: Trivially configured `nginx` nodes serving different static files to allow
    differentiation between the nodes.

Each of these containers is added to a bridge network for the application. These containers can reach
each other by name (`web[123]` and `proxy`), but from the host, we cannot connect to them,
except via the `docker` command.

## Prerequisites

Throughout, we assume that you are running on a modern Linux distribution. The author uses the RHEL 
family of operating systems - CentOS Streams, AlmaLinux, Rocky Linux, Oracle Linux, but any
platform-specific commands will be flagged. If you are using a Debian-family distribution, you will
need to use an equivalent on those platforms.

We also assume that you have installed Docker (docker.io) and
[docker-compose](https://github.com/docker/compose/releases). The instructions should work with
podman and podman-compose, but we have not tested them.

We will also use `git` to check out the code from github.

We will want to run docker-compose as an unprivileged user, so we have to add our user to
the `docker` group.

    usermod -aG docker $USER

After running this, you need to ensure that you relaunch a login shell for the change to take effect.
You can do this explicitly with:

    sudo -s -u $USER

but for the change to take effect permanently in your instance, you will need to restart the `sshd`
service. On the RHEL family, this means:

    sudo systemctl restart sshd

The next time you connect to your instance with ssh, you should see `docker` group in the output of
`groups` for your user.

To test that you now have Docker correctly installed and configured, run:

    docker run hello-world

You should see a Docker container being downloaded from dockerhub, and a message confirming that your
installation appears to be running correctly.

## Getting started

Now that we have things set up, check out this project:

    git clone https://github.com/dneary/nginxlb.git
    cd nginxlb

We should now be ready to run:

    docker-compose up -d

This will process the docker-compose.yml file in this directory, download the `nginx` container, and start
four copies of it, one node called `proxy` with the config in `nginx/proxyconf.d`, and three with the
identical trivial config in `nginx/conf.d`. After running `docker-compose`, if all goes well, you should
now be able to go to `http://localhost:80` and see which servers you are connecting to.
