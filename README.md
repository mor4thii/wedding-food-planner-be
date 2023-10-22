# Wedding Food Planner Backend

## Local Development Environment

### Installing Prerequisites on MacOS

We use [Multipass](https://multipass.run/docs/installing-on-macos) from Canonical together
with [microk8s](https://microk8s.io/docs/install-multipass) to spin up a local k8s instance to deploy our application
and its dependencies to. Shout out to [kubajz.dev](https://kubajz.dev/replace-docker-desktop-with-multipass) for the
tutorial on this.

#### Multipass

Install Multipass using [Homebrew](https://brew.sh/).

```shell
brew install --cask multipass
```

Prepare the cloud-config file to use and spin up a VM using Multipass.

```shell
cp local-k8s.yml.template local-k8s.yml
# Enter your public SSH key under `ssh-authorized-keys`
multipass launch --name microk8s-vm --memory 4G --disk 40G --cloud-init local-k8s.yml
```

After the VM was launched, test it with:

```shell
multipass list
multipass exec microk8s-vm  -- docker --version
```

#### Microk8s

When everything is set up, install microk8s.

```shell
brew install ubuntu/microk8s/microk8s
```

Verify that we have microk8s up and running with:

```shell
microk8s status
```

To use the microk8s cluster, we need to provide valid configuration files for microk8s' bundled `kubectl` as well as the
default `kubectl` command if we have it.

```shell
mkdir -p ~/.microk8s && microk8s config > ~/.microk8s/config
mkdir -p ~/.kube && microk8s config > ~/.kube/config
```

Verify that everything worked by running:

```shell
microk8s kubectl get pods
```

#### Docker CLI Setup

We are not using Docker Desktop for this project, but rather Multipass with a custom docker context.
To start, make sure the docker CLI and the buildx plugin are installed and properly configured.

```shell
brew install docker docker-buildx
mkdir -p ~/.docker/cli-plugins
ln -sfn /usr/local/opt/docker-buildx/bin/docker-buildx ~/.docker/cli-plugins/docker-buildx
```

Get your VM IP by checking `multipass list`. Then, create a docker context as follows.

```shell
docker context create multipass \
  --description "Docker Engine in Multipass" \
  --docker "host=ssh://ubuntu@<your-VM-ip>"
docker context use multipass
```

Make sure to connect to your VM using ssh at least once before trying to build docker images and add it
to `known_hosts`.

```shell
ssh ubuntu@<your-VM-ip>
```

When you run, for example, `docker build .` now, it should try to build an image but fail as there is no Dockerfile in
the repository root.
