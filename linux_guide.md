# PINLab Technical Guide

## Connecting to the workstations

### VPN

To access the workstations from a non-sapienza network you have to use a VPN that you can download and setup by following the guide at [this link](https://web.uniroma1.it/infosapienza/servizio-vpn-di-ateneo).

After installing the VPN, connect to this server: `castore.uniroma1.it` and use your sapienza credentials to login (_name.surname@uniroma1.it_).

### SSH

#### Passwd-free connection

You can connect to all the machines available via ssh. In order to avoid inserting the password at each connection, you can set-up an ssh-key which allows you to connect directly, without a password. To accomplish it, follow the steps below.

**If you are using windows** you should perform these steps with the [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) since some of the bash commands needed are available just on linux.

1. Create a ssh key on your laptop if you don't have one:
```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

2. Copy public key to the workstation:
```sh
ssh-copy-id -i ~/.ssh/id_ed25519.pub REMOTE_HOST
```

You can setup a shortcut to connect to each machine by adding the following lines to `~/.ssh/config`:  
```
Host HOSTNAME
  HostName IP_ADDRESS
  User USERNAME
```
`HOSTNAME` is a name of your choice for each host-user configuration;  
`IP_ADDRESS` is the IP address of the machine (e.g., 165.234.bla.bla);  
`USERNAME` is the name of your user on the host machine (e.g., escobar).


## Create user

```sh
# create user with name USERNAME (e.g. paolo)
# GROUP_NAME corresponds to the name of the pc (e.g. ares)
sudo useradd -g GROUP_NAME -G sudo -m -s /bin/bash USERNAME

# set user password in order to be able to access
sudo passwd USERNAME

# remember to change the host in your ssh command with the new username
```


## Conda

### Installing miniconda

With main user of the pc (e.g., ares, dodo, zeus) do:
```sh
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh
rm ~/miniconda.sh
sudo chgrp -R GROUP PATH_TO_CONDA
sudo chmod 770 -R PATH_TO_CONDA
```

On normal user do:
```sh
export PATH="PATH_TO_CONDA/bin:$PATH"
exec bash
conda init
exec bash
```

### Cloning old environments

Due to changes in the path of the conda environments, copy and pasting the old env folder into the new directory does not fully work.
The best solution is to export the old environment into a `yaml` file and then creating a new one.

Export and remove the old environment:
```sh
conda activate ENV_NAME
conda env export --no-builds > env.yml
conda remove --name ENV_NAME --all
```

Create the new environment:
```sh
conda env create -f env.yml
```

## Setting default permissions (tentative)

The following process should be executed on the shared folder which all users should be able to access (e.g. `/storage/hdd-1` on ares). Moreover, we need to exec this just one time and then all the files and folders created in the shared directory should be automatically accessible (and writable) by everyone.

Before starting, make sure that `acl` is installed by running: `sudo apt-get install acl`.

1. Set the `setgid` bit, so that files/folders under <directory> will be created with the same group as <directory>:  
    ```sh
    chmod g+s <directory>       # for example /storage/hdd-1
    ```
    g+s will ensure that new content in the directory will inherit the group ownership
  
2. Set the default ACLs for the group and other:  
    ```sh
    setfacl -R -m g::rwx <directory>      # give read, write and execution rights to users inside the group
    setfacl -R -m o::rx <directory>       # give read and execution rights to users outside the group
    ```
  
3. Verify that everything has been set correctly:  
    ```sh
    getfacl <directory>
    ```  
    Output:
    ```sh
    # file: ../<directory>/
    # owner: <user>
    # group: media
    # flags: -s-
    user::rwx
    group::rwx
    other::r-x
    default:user::rwx
    default:group::rwx
    default:other::r-x
    ```
