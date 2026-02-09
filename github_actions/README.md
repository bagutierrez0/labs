# GitHub Actions

## Install dependencies
- curl
- tar
- git

## Create runner user
```bash
sudo adduser --disabled-password --gecos "" gha
sudo mkdir -p /opt/actions-runner
sudo chown -R gha:gha /opt/actions-runner
```

## Run the following commands as gha (user)
```bash
sudo -u gha -i
cd /opt/actions-runner
```

## Get the runner setup commands from github
Go to repo settings.
