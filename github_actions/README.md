# GitHub Actions


# Regular Runner Setup

## Setup
These are the steps you need to take to let the server and GitHub communicate and utilize GitHub Actions.

### 1. Install Dependencies
```bash
sudo apt update
sudo apt install -y curl tar git
```

### 2. Create a dedicated runner user
```bash
sudo adduser --disabled-password --gecos "" gha
sudo mkdir -p /opt/actions-runner
sudo chown -R gha:gha /opt/actions-runner

# swtich to the gha user
sudo -u gha -i
cd /opt/actions-runner
```

### 3. Get the runner setup commands from GitHub
- go to repo settings
- look at code and automation side tab
- click actions
- click runners
- click new self-hosted runner
- go through the setup steps (choose runner image for your os, os architecture)
- run the commands on the server

### 4. Verify the service is running
```bash
sudo systemctl status actions.runner.*
```

### 5. Clone the repo that you set up GitHub Actions for onto the server
Remember that wherever you clone the repo, the gha user needs the proper permissions. 
```bash
cd <cloned-repo-path>
```

### 6. Add the GitHub Actions workflow file
Add this to `.github/workflows/test.yml` to the root of the cloned repo directory. The `run:` section is what will run when the repo is modified. This will appear in the repo's Actions tab. You can see things in real-time. A good use case is to pull the latest changes from a repo when they are available. This is what I am using this for in my projects.

```yaml
name: Test self-hosted runner

on:
  push:
    branches: ["main"]

jobs:
  test:
    runs-on: self-hosted
    steps:
      - run: |
          echo "Runner hostname:"
          hostname
          echo "Whoami:"
          whoami
          git fetch origin
          git reset --hard origin/main
          git rev-parse HEAD
```

## Notes
- It does not matter where the cloned repo is on the filesystem as long as this runner is configured for the repo on GitHub's website in the repo settings and the server has the workflow. The part you need to  keep in mind is the permissions aspect of the gha user, you  need to ensure that whatever commands you run in the `run:` section within the workflow, that the gha user has the proper permissions to do that.

- The `.github` directory must be in the root of the repo, otherwise this will not work properly

- You can use an SSH deploy instead of a runner, just create a user and a new key pair for communications between the server and github, this is a safer alternative

# SSH Runner

## Setup

### 1. Create Deploy User On The Server
Note you may need to add the deploy user to a user group (for a particular service/tool). Depends on what you are doing.
```bash
sudo adduser --disabled-password --gecos "" deploy
sudo usermod -aG www-data deploy
```

### 2. Create SSH keypair for GitHub Actions To Use
```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f gha_deploy_key
```

### 3. Install The Public Key On The Server
```bash
sudo mkdir -p /home/deploy/.ssh
sudo chmod 700 /home/deploy/.ssh
sudo tee -a /home/deploy/.ssh/authorized_keys < gha_deploy_key.pub
sudo chown -R deploy:deploy /home/deploy/.ssh
sudo chmod 600 /home/deploy/.ssh/authorized_keys
```
### 4. Add Secrets To GitHub
- SSH_HOST: server IP/hostname
- SSH_user: deploy
- SSH_KEY: private key

### 5. Clone The Repo
```bash
git clone <repo-url>
```

### 6. Add The Workflow File Into The Cloned Repo
```bash
name: Deploy via SSH

on:
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            # run whatever commands you need to run when changes
            # within the repo are made
            cd <dir>
            git fetch origin
            git reset --hard origin/main
```

## Notes
- Again remember the deploy user will need the proper permissions to run whatever commands you need to be run in `run:` when changes are noticed.