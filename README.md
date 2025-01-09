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

```bash
export ANSIBLE_STDOUT_CALLBACK=yaml
ansible-playbook play_install_gitlab.yml
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