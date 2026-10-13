# Deployment demo

This deployment builds IT-Tools from this fork, runs it in Nginx, and exposes port 8080.
GitHub stores the source. SSH transfers and manages the deployment.
No GitHub token or SSH private key is stored on the server.

## Direct deployment when the server can reach GitHub

Install Git, Docker Engine, and the Docker Compose plugin first.

```sh
git clone https://github.com/ziran258/it-tools.git repo
cd repo
git rev-parse HEAD
export SOURCE_REV="$(git rev-parse HEAD)"
export WEB_PORT=8080
sudo --preserve-env=SOURCE_REV,WEB_PORT docker compose -f deploy/compose.yaml config --quiet
sudo --preserve-env=SOURCE_REV,WEB_PORT docker compose -f deploy/compose.yaml up -d --build
curl --fail http://127.0.0.1:8080/
```

## Deployment when the server cannot reach GitHub

On a computer that can reach GitHub, create a complete Git bundle from the fork:

```sh
git clone https://github.com/ziran258/it-tools.git repo
git -C repo bundle create ../it-tools.bundle --all
```

Transfer the bundle over SSH to an independent deployment directory.
On the server, clone the bundle and restore the GitHub origin:

```sh
git clone it-tools.bundle repo
git -C repo remote set-url origin https://github.com/ziran258/it-tools.git
cd repo
export SOURCE_REV="$(git rev-parse HEAD)"
export WEB_PORT=8080
sudo --preserve-env=SOURCE_REV,WEB_PORT docker compose -f deploy/compose.yaml up -d --build
```

This preserves the fork's commit history. Future updates can use a new bundle transferred
over SSH, or normal git fetch when GitHub connectivity is restored.
Code pushes to GitHub do not automatically update the deployed service.

## Network and validation

Allow TCP 8080 in the instance's cloud security group or Lighthouse firewall.
Open http://SERVER_PUBLIC_IP:8080 from an external browser.
Verify a tool works, refresh its direct URL, then restart only the demo container and
verify it recovers. The Docker daemon should be enabled at boot.

```sh
sudo docker compose -f deploy/compose.yaml ps
sudo docker compose -f deploy/compose.yaml logs --tail=100
sudo docker compose -f deploy/compose.yaml restart web
```

For every Compose command, keep SOURCE_REV set to the deployed commit, or save
SOURCE_REV and WEB_PORT in deploy/.env and use --env-file deploy/.env.

The build pins pnpm to the package.json version and both base images by digest.
Vite uses the upstream 4096 MB JavaScript heap limit. For a small server, build
on a larger machine or provide enough temporary swap during compilation.
The running Nginx container needs far less memory than the build process.
