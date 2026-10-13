# Linux deployment

This fork contains the source and deployment configuration for an IT-Tools demo.
The default deployment uses the project's official release image, pinned by version
and SHA-256 digest. It does not build a custom image from the fork.

## Deploy the official release

Requirements: Git, Docker Engine, Docker Compose, and an available host port.

```sh
git clone https://github.com/ziran258/it-tools.git repo
cd repo
sudo docker compose -f deploy/compose.release.yaml config --quiet
sudo docker compose -f deploy/compose.release.yaml up -d
curl --fail http://127.0.0.1:8080/
```

Application version: 2024.10.22-7ca5933.
Official release: https://github.com/CorentinTh/it-tools/releases/tag/v2024.10.22-7ca5933

When the server cannot reach GitHub, create a complete Git bundle on another
computer, transfer it over SSH, and clone it on the server:

```sh
git clone https://github.com/ziran258/it-tools.git repo
git -C repo bundle create ../it-tools.bundle --all
# Transfer it-tools.bundle to the server using scp, then on the server:
git clone it-tools.bundle repo
git -C repo remote set-url origin https://github.com/ziran258/it-tools.git
cd repo
sudo docker compose -f deploy/compose.release.yaml up -d
```

This preserves Git history and the original GitHub origin without placing a GitHub
token or SSH private key on the server. Code changes pushed to GitHub do not
automatically update the deployed service.

## Validate and operate

Allow TCP 8080 in the cloud instance's security group or Lighthouse firewall.
Visit http://SERVER_PUBLIC_IP:8080 from an external browser, test a tool, and
refresh its direct URL. To use another host port, set WEB_PORT.

```sh
sudo docker compose -f deploy/compose.release.yaml ps
sudo docker compose -f deploy/compose.release.yaml logs --tail=100
sudo docker compose -f deploy/compose.release.yaml restart web
sudo docker compose -f deploy/compose.release.yaml stop web
```

The service restarts automatically unless explicitly stopped. Enable the Docker
daemon at boot.

## Optional source build

deploy/Dockerfile and deploy/compose.yaml can build a custom image from this fork.
Both base images are pinned by digest and pnpm is pinned to 9.11.0.
Run on a build machine with enough memory: the upstream Vite build allows a
4096 MB JavaScript heap, and a 2 GB server may spend a long time swapping.

```sh
export SOURCE_REV="$(git rev-parse HEAD)"
sudo --preserve-env=SOURCE_REV docker compose -f deploy/compose.yaml up -d --build
```

Use either compose.release.yaml or compose.yaml for the service, and keep using
the selected file for all subsequent commands.
