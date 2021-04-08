# things-i-forget
Commands and other things I Google all the time with varying degrees of success. Here I'll put the stuff I want to remember.

#### The things:
- [Git](#Git)
- [Docker](#Docker)
- [AWS](#AWS)
- [Bash/Zsh Terminal](#Terminal)
- [Python](#Python)
- [VS-Code](#VS-Code)
- [Mac stuff](#MacOS)

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

## Terminal
Put this in your `.bashrc` or `.zshrc` to alias command for jupyter notebook server. This is more useful for running in Docker since that command requires more flags that I forget.
```bash
# add jupyter command
alias notebookserver='jupyter notebook --allow-root --ip=0.0.0.0 --no-browser --port 9999'
```

Open a file with Sublime
```bash
open -a /Applications/Sublime\ Text.app <filename>
```

Remove all versions from a pip requirements file:
```bash
cat requirements.txt | sed 's/=.*//' > requirements-no-versions.txt
```

Sometimes the terminal slows down on MacOS, and removing these Apple System Logs can speed it back up. For example, launching a Python virtual env was getting noticeably slower and after running this command, it sped back up:
```bash
sudo rm -rf /private/var/log/asl/*.asl
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
# OPTIONAL: if you want to install local module you're working on
poetry install
```
The last command can get fancier including: separating dev and prod deps and `grep`-ing certain lines or trimming off package version numbers.

Have poetry create the virtual environments in your project's folder (like `venv` does) so it's easier for VSCode to find, since I sometimes have trouble getting VSCode to use the right python interpreter:
```bash
poetry config virtualenvs.in-project true
```

#### Conda
[This SO answer](https://stackoverflow.com/a/58045984/9448289) walks through how to install and use Anaconda _alongside_ `pyenv` on MacOS. Sometimes, I work on projects that have dependencies outside of python which Conda can handle well – e.g. `pyarrow`.

```bash
# brew install anaconda
brew cask install anaconda
```

Then, pay attention to where brew installs conda on your system. Note that path to use in next command:
```bash
source <path to conda>/bin/activate
conda init zsh
# disable init of env "base"
conda config --set auto_activate_base false
```

**Side note**: If commands above say something about no write permission on a conda file in `~/.conda/`, run this:
```bash
sudo chmod -R 775 ~/.conda/
```

Then you should be able to create env per usual commands:
```bash
# virtual environments from conda
conda create -n py37 python=3.7
conda env list
conda activate py37
conda deactivate
```

Conda install requirements from pip's requirements.txt (also see bash above to remove version numbers if necessary), in an activated environment:
```bash
conda install --force-reinstall -y -q --name py37 --file requirements-no-versions.txt
```

Install local package you're working on in the virtual env:
```bash
# install, like pip install -e .
conda develop .
# uninstall
conda develop -u .
```

#### Jupyter
Enable notebook extensions - there's mostly wrong info on StackOverflow and Github issues out there. THIS is it:
```bash
pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
jupyter nbextension enable varInspector/main
```


## VS-Code

Setup **`black`** format on save:
- Go to Settings
- Search for "format on save", and check the box
- Search for "python formatting provider", from drop down select the library you want.

Configuring **YAPF**:
In user or project settings.json, you can configure YAPF by adding args like,
```json
"python.formatting.yapfArgs": ["--style={ based_on_style: pep8, column_limit: 120 }"],
```

## MacOS

Permanently show hidden files in finder (which you would think would be a setting you could change via somewhere besides only the terminal):
```bash
defaults write com.apple.Finder AppleShowAllFiles true
killall Finder
```
Same command but `true` -> `false` hides the files again.
