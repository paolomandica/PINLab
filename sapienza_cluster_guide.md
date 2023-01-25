# Sapienza Cluster Unofficial Guide

**Official guide**: [gdoc link](https://docs.google.com/document/d/1XY7cy8Roqq4w_Q-zNiVTgHXwyRZn9WhqiUMyCky3514)

## SSH setup guide

Replace `user` with your username (e.g., *pmandica*)

```bash
# First connection
ssh -J user@151.100.174.45 user@submitter

# Generate a ssh key if you don't have one
# then, start ssh-agent and add ssh key
# run everything from .ssh directory
ssh-keygen -t ed25519 -f id_ed25519
eval "$(ssh-agent -s)"
ssh-add ./id_ed25519

# ssh-copy-id to avoid password input at each connection
ssh-copy-id -i id_ed25519.pub -o ProxyJump=user@151.100.174.45 user@submitter
```

SSH config hosts:  
```ssh
Host cluster-front
  HostName 151.100.174.45
  User USERNAME

Host cluster
  HostName submitter
  User USERNAME
  ProxyJump cluster-front
```

## Managing jobs

```ssh
# Check job(s) status
condor_q
watch -n 0.5 condor_q

# Submit a job
condor_submit docker.sub

# Stop and remove a job
condor_rm JOB_ID

# Attach to job via ssh
condor_ssh_to_job JOB_ID
```

### Docker jobs

Example of `docker.sub` file for job submission:
```ssh
universe                = docker
docker_image            = paolomandica/ubuntu-18.04-cluster	# docker image
executable              = /bin/bash
arguments               = /home/pmandica/condor/temp_sh.sh	# script to run
output                  = logs/out.$(ClusterId).$(ProcId)
error                   = logs/err.$(ClusterId).$(ProcId)
log                     = logs/log.$(ClusterId).$(ProcId)
request_cpus            = 8
# request_memory          = 1G
request_gpus            = 1
# requirements = (TARGET.Machine == "node113")
+MountData1=TRUE
+MountData2=FALSE
+MountHomes=TRUE
queue 1
```

