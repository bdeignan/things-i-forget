# things-i-forget
Commands and other things I Google all the time with varying degrees of success. Here I'll put the stuff I want to remember.

#### The things:
- [Git](#Git)
- [Docker](#Docker)
- [AWS](#AWS)
- [Bash](#Bash)
- [Python](#Python)
- [VS-Code](#VS-Code)

## Git
[This link](http://gwu-libraries.github.io/Git.html) is helpful.

```bash
# git unstage a file you added
git reset HEAD FILE-TO-UNSTAGE
```

Checkout remote branch and rename it (sometimes collaborators create awfully long branch names)
```bash
git checkout -b mybranch origin/myremotebranch
```

One-line git log
```bash
git log --pretty=oneline
```

Make your default init branch "main" (Only works for Git 2.28+)
```bash
git config --global init.defaultBranch main
```

## Docker
Create new docker container with local working directory mounted and port binding (e.g. for jupyter notebooks and more)
```bash
# starts an interactive session
docker run -it -v ${PWD}:/Docker -p 8888:8888 -p 4040:4040 --name CONTAINERNAME IMAGENAME
```

Docker restart container (run without `-ai-` to start without interactive session)
```bash
# interactive
docker start -ai CONTAINERNAME
```

Connect to running container that has bash installed
```bash
docker exec -it CONTAINERNAME /bin/bash
```

**Disconnect** from container _without_ shutting container down.
` ctrl+p` then `ctrl+q`


Running **Jupyter notebook** in container on specific port (not specific to Docker, this is just a typical use case for me)
```bash
# Can alias this command in .bashrc, eg as "notebookserver"
jupyter notebook --allow-root --ip=0.0.0.0 --no-browser --port 9999
```

**Docker compose** to build first time. Leave `--build` off after first time. The `-d` flag is important. It tells docker to start containers in the background and leave them running.
```bash
# is a wd with docker-compose.yml
docker-compose up -d --build
# shutdown containers
docker-compose down
```

Truncated docker `ps` check:
```bash
docker ps | less -S
```

## AWS
Copy files from s3 to local file system. `--profile` optional depending on if you're using that to authenticate.
```bash
aws --profile <PROFILENAME> s3 cp --recursive s3://bucket/FOLDER /LOCAL/FOLDER/PATH
```

This is pretty obscure, but ECR login method changed significantly for AWS-CLI 2.x. This is how I logged into ECR with Docker to pull images locally.
```bash
aws --profile <PROFILENAME> ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

## Bash
Put this in your `.bashrc` to alias command for jupyter notebook server. This is more useful for running in Docker since that command requires more flags that I forget.
```bash
# add jupyter command
alias notebookserver='jupyter notebook --allow-root --ip=0.0.0.0 --no-browser --port 9999'
```

Open a file with Sublime
```bash
open -a /Applications/Sublime\ Text.app <filename>
```

## Python

#### Pyenv
Don't forget to add `pyenv init` to .bashrc or .zshrc
```bash
eval "$(pyenv init -)"
```

#### Poetry
Initalize a project in an existing directory:
```bash
poetry init
```

Otherwise, from scratch:
```bash
# create a new project and directory called "project"
poetry new project
cd project
# install pandas
poetry add pandas
# run a python command in environment
poetry run python <script.py>
# activate environment
poetry shell
```

Delete an existing poetry virtualenv:
```bash
# This will give you the name to remove in next command
poetry env list
# Replace WhatEvs env below with the actual name
poetry env remove whatever-WhATeVs-py3.9
```

Get poetry project/virtualenv information
```bash
poetry env info
```
For example the `Path: {}` info can be used for settings the interpreter in VS Code.

How to create a virtualenv from a `requirements.txt` file in existing project:
```bash
# initialize env and follow prompts
poetry init
# install reqs
cat requirements.txt|xargs poetry add
```
The last command can get fancier including: separating dev and prod deps and `grep`-ing certain lines or trimming off package version numbers.

## VS-Code

Setup `black` format on save:
- Go to Settings
- Search for "format on save", and check the box
- Search for "python formatting provider", from drop down select the library you want.
