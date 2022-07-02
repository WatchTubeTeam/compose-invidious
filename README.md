# compose-invidious

## Setup

### Setting up Docker

#### Install Docker and Docker Compose v2

See ["Install Docker Engine on Ubuntu"][docker-install-guide] for Ubuntu. Here's
a copy of the Ubuntu instructions listed there:

[docker-install-guide]: https://docs.docker.com/engine/install/ubuntu/

```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Note that this guide installs the package named `docker-compose-plugin`. This is
important; it's Docker Compose itself. If you get the following error:

```
docker: 'compose' is not a docker command.
See 'docker --help'
```

try installing Docker Compose:

```
sudo apt install docker-compose-plugin
```

#### Add your user to the `docker` group

```
sudo usermod -aG docker [username]
```

### Setting up `compose-invidious`

#### Clone the repo

```
sudo mkdir /opt/compose-invidious
sudo chown [username]:[username] /opt/compose-invidious
git clone https://github.com/WatchTubeTeam/compose-invidious.git /opt/compose-invidious
```

#### Configure Invidious

Make a copy of the example config.

```
cd /opt/compose-invidious
cp .env.example .env
```

The example config has the default settings shown, commented out. Here's an
explanation of each setting:

- `TAG`: the version of Invidious to use. Use `latest` for amd64/x86_64
  machines, and `latest-arm64` for arm64 machines. This defaults to
  `latest-amd64` since the majority of our servers run on Oracle Cloud Ampere.
- `INSTANCE_ID`: the first part of the domain, for example: the `uk1` in
  `uk1.watchtube.app`. **This setting is REQUIRED**
- `PORT`: the port that Invidious should run on. This port is only accessible
  locally and should be reverse proxied. This option lets you change it to
  prevent it from conflicting with other software running on the same machine.

#### Start the server

```
./update.sh
```

This simple script makes it easy to update the Invidious instance. All it does
is pull the latest version of this repo, pull the latest Docker image, and then
start the containers.

Read the script, it's only several lines long.

### Setting up a reverse proxy

This repo does not set Invidious up to be production-ready. It still needs a
reverse proxy. If you are setting up `compose-invidious` on a machine where
you're already running other websites (and a reverse proxy), use that existing
one. Otherwise, here's how to set up Caddy for a simple setup.

#### Install Caddy

See the [Debian/Ubuntu/Raspbian installation guide][caddy-install-guide]. Here's
a copy of the instructions listed there:

[caddy-install-guide]: https://caddyserver.com/docs/install#debian-ubuntu-raspbian

```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

#### Add the domain to Cloudflare

Ideally, you should add the domain to Cloudflare before setting up the reverse
proxy. That means that the domain will have propagated already and it will be
able to fetch the HTTPS certificate immediately.

However, don't worry if you aren't able to right away, Caddy will automatically
try again periodically. If you know the domain has already propagated, you can
restart Caddy and it will try again instantly:

```
sudo systemctl restart caddy
```

#### Add configuration to Caddy

Put the following contents in `/etc/caddy/Caddyfile`:

```caddyfile
[instance_id].watchtube.app {
    reverse_proxy localhost:81
}
```

#### Reload Caddy's configuration

```
sudo systemctl reload caddy
```
