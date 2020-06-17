# things-i-forget
Commands and other things I Google all the time with varying degrees of success. Here I'll put the stuff I want to remember.

#### The things:
- [Git](#Git)
- [Docker](#Docker)
- [AWS](#AWS)

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
