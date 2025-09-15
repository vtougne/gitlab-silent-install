## Table of Contents
- [Prerequisites](#prerequisities)
  - [Set DNS entry](#set-dns-entry)
    - [Install from Windows/WSL or Linux host to remote Linux host](#install-from-windowswsl-or-linux-host-to-remote-linux-host)
    - [Install on localhost](#install-on-localhost)
- [Install](#install)
- [Connect](#connect)
  - [Get password](#get-password)
  - [Init password](#init-password)

## Prerequisities

### set DNS entry ...
#### ... from Windows/WSL or Linux host to remote Linux host

Ensure that the following aliases are set in your DNS, otherwise declare them in hosts files.
  > note 1: a dns suffix is mandatory (me in the follwoing example):
  > note 2: a dns suffix is mandatory (me in the follwoing example):

Exampe for a given host named my_host
```
# Windows: C:\Windows\System32\drivers\etc\hosts
# Linux: /etc/hosts

192.168.XXX.XXX      my_host    my_host.me     registry.my_host.me
```

#### ... from  localhost


```
# Windows: C:\Windows\System32\drivers\etc\hosts
# Linux: /etc/hosts
127.0.0.1       local-gitlab.me     registry.local-gitlab.me
```

## Install

### Local Installation

For local GitLab installation:

```bash
export ANSIBLE_STDOUT_CALLBACK=yaml
ansible-playbook -i inv_local-gitlab.yml play_install_gitlab.yml --ask-become-pass
```

### Remote Installation

For remote server installation:

```bash
export ANSIBLE_STDOUT_CALLBACK=yaml
ansible-playbook -i inv_remote.yml play_install_gitlab.yml --ask-become-pass
```

### Troubleshooting sudo issues

If you encounter `sudo: a password is required` errors, you have two options:

1. **Use --ask-become-pass flag** (recommended for security):
   The commands above include this flag to prompt for your sudo password.

2. **Configure passwordless sudo for Docker** (less secure):
   ```bash
   echo "$USER ALL=(ALL) NOPASSWD: /usr/bin/docker" | sudo tee /etc/sudoers.d/docker-nopasswd
   ```
   Then run without --ask-become-pass:
   ```bash
   ansible-playbook -i inv_local-gitlab.yml play_install_gitlab.yml
   ```


## Connect
### Get password
```
sudo docker exec -it local-gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

### init password
```
read password
docker exec -ti local-gitlab bash -c "gitlab-rake 'gitlab:password:reset[root]'<<EOF
${password}
${password}
EOF"
```

### Check available GitLab versions
```bash
curl -s "https://registry.hub.docker.com/v2/repositories/gitlab/gitlab-ce/tags/?page_size=10" | grep -o '"name":"[^"]*"' | head -10
```

## Check current version
```bash
docker exec local-gitlab gitlab-rake gitlab:env:info
docker exec local-gitlab gitlab-rake gitlab:env:info | grep "^Version" | head -1
```

## Check current version
```bash
docker exec local-gitlab head -1 /opt/gitlab/version-manifest.txt
gitlab-ce 17.7.0
```
