# solita/ansible-ssh

A Docker image with [Ansible](https://www.ansible.com/) and an SSH server.

## Supported tags

* `latest`, `2.0.2.0`
* `2.0.0.2`

## But why?

You can't do much with Ansible unless you can access your servers. For that, you need your SSH keys to be loaded in an [ssh-agent](https://en.wikipedia.org/wiki/Ssh-agent) accessible within the container. The easiest way to do that is to use your local `ssh-agent`, and SSH into the container with agent forwarding enabled.

## Authorizing SSH logins

The `solita/ansible-ssh` base image contains a user called `ansible`, but does not allow anyone to log in. Use a `Dockerfile` such as the following to create an image that allows logging in as `ansible` with your SSH keys (assuming that you have copied your public key `id_rsa.pub` to the same directory as the `Dockerfile`).

    FROM solita/ansible-ssh
    COPY id_rsa.pub /tmp/
    RUN cat /tmp/id_rsa.pub >> /home/ansible/.ssh/authorized_keys && rm -f /tmp/id_rsa.pub

## Running

This image is based on [`solita/ubuntu-systemd-ssh`](https://hub.docker.com/r/solita/ubuntu-systemd/), so you need to add a couple of flags to the `docker run` command to make `systemd` play nice with Docker. Additionally you should forward the SSH port `22` to some port on the Docker host. Here's a full command line for starting a container from `YOUR_IMAGE` with the SSH server's port forwarded to `2222` on the Docker host.

    docker run -d -p 22:2222 --stop-signal=SIGRTMIN+3 --tmpfs /run --tmpfs /run/lock -v /sys/fs/cgroup:/sys/fs/cgroup:ro YOUR_IMAGE

## Connecting to the container

You can SSH into the container as the user `ansible`. Use the `-A` switch to enable agent forwarding.

Assuming your Docker host is `localhost` and you've forwarded the container's port `22` to `2222` on the host, you can connect to the container like this:

    ssh \
      -o UserKnownHostsFile=/dev/null \
      -o StrictHostKeyChecking=no \
      -A \
      -p 2222 \
      ansible@localhost

The `-o` switches disable host key checking. Without them, SSH will refuse to connect when you use the port `2222` for another container with a different host key. **You should only disable host key checking when there is no chance of a man-in-the-middle attack** (e.g. when you're connecting to a container running on the same physical host)!

The `-A` switch enables agent forwarding.

## License

Copyright © 2016 [Solita](http://www.solita.fi). Licensed under [the MIT license](LICENSE).
