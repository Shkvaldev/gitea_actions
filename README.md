# Gitea + CI/CD
> Guide on installing and using it

## Questions

- What for?
>Using CI/CD increases development speed and makes it more easy yet. Thats why you should use CI/CD in development.

- Why Gitea?
> This is the golden mean between lightness and functionality

- What we are goin to use as 'worker' as CI/CD agent?
> Official Gitea runner will be used since it is very functional and it's action's syntax similar to Github Actions (some would prefer using Drone, but it is outdated - I don't like it)

## Installation
> We will set up our environment in Ubuntu 22.04

1. Install Docker:
```bash
sudo apt update -y && sudo apt upgrade -y

sudo apt install curl gnupg lsb-release software-properties-common ca-certificates apt-transport-https -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y
sudo apt install docker-ce -y
```

2. Enable Docker:
```bash
sudo systemctl enable --now docker
```

3. Install Gitea in Docker:
```bash
sudo docker run -d --name gitea \
    -p <HOST PORT>:<CONTAINER PORT> \
    -p 222:22 \
    -v gitea_data:/data \
    -e USER=git \
    -e GITEA__database__DB_TYPE=sqlite3  \
    -e GITEA__database__PATH=/data/gitea/gitea.db \
    -e GITEA__server__ROOT_URL=<YOUR URL HERE> \
    -e GITEA__server__SSH_PORT=22 \
    -e GITEA__server__DOMAIN=localhost \
    -e GITEA__server__HTTP_PORT=<HTTP PORT> \
    -e GITEA__server__DISABLE_SSH=true \
    -e GITEA__log__MODE=console \
    gitea/gitea:latest
```

4. After installing via browser - navigate to admin panel > Actions and get from it token for actor

5. Generate config for actor:
```bash
sudo docker run --entrypoint="" --rm -it gitea/act_runner:latest act_runner generate-config > config.yaml
```

6. Install actor:
```bash
sudo docker run -d --name runner \
    -v $PWD/config.yaml:/config.yaml \
    -v $PWD/data:/data  \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e CONFIG_FILE=/config.yaml \
    -e GITEA_INSTANCE_URL=<YOUR URL> \
    -e GITEA_RUNNER_REGISTRATION_TOKEN=<RECEIVED TOKEN> \
    -e GITEA_RUNNER_NAME=<SOME NAME FOR RUNNER> \
    gitea/act_runner:nightly
```

Well done! Now you can verify actor existance in Gitea admin panel

## Actions

To use actions - you must create **.gitea/workflows** in your project root and place .yaml files in it.
There are some examples in **actions/** dir for FastAPI python app and Vue JS project (with Dockerfiles and other)
