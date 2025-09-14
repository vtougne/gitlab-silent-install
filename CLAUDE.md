# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an Ansible-based GitLab installation automation project that provides silent installation of GitLab CE/EE with Docker containers. The project supports both local and remote installations with automatic GitLab Runner setup and container registry configuration.

## Architecture

The project uses a main Ansible playbook (`play_install_gitlab.yml`) with two inventory configurations:
- `inv_local.yml`: For localhost installations using ARM-based GitLab image
- `inv_remote.yml`: For remote server installations using standard GitLab CE image

Key components:
- **GitLab Container**: Main GitLab instance with web interface on port 80/443
- **GitLab Runner**: Containerized CI/CD runner with Docker executor
- **Container Registry**: SSL-enabled private Docker registry integrated with GitLab
- **SSL Certificates**: Self-signed certificates auto-generated for registry

## Installation Commands

### Prerequisites
Ensure DNS/hosts entries are configured as documented in README.md.

### Install GitLab (Local)
```bash
export ANSIBLE_STDOUT_CALLBACK=yaml
ansible-playbook -i inv_local.yml play_install_gitlab.yml
```

### Install GitLab (Remote)
```bash
export ANSIBLE_STDOUT_CALLBACK=yaml
ansible-playbook -i inv_remote.yml play_install_gitlab.yml
```

### Common Post-Installation Commands

Get initial root password:
```bash
sudo docker exec -it local-gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

Reset root password:
```bash
read password
docker exec -ti local-gitlab bash -c "gitlab-rake 'gitlab:password:reset[root]'<<EOF
${password}
${password}
EOF"
```

Test Docker registry login:
```bash
echo 'PASSWORD' | docker login registry.HOSTNAME.me --username root --password-stdin
```

## Key Variables

The playbook uses these important variables defined in inventory files:
- `gitlab_location`: "local" or "remote" - determines execution delegation
- `gitlab_image`: GitLab Docker image to use
- `gitlab_instance_fqdn`: Fully qualified domain name for GitLab instance
- `gitlab_registry`: Registry hostname (registry.HOSTNAME.me)
- `gitlab_ssh_port_mapping`: SSH port mapping for GitLab container

## Installation Process

The playbook performs these major steps:
1. Validates DNS/network connectivity to GitLab FQDN and registry
2. Creates Docker Compose configuration with GitLab and GitLab Runner services
3. Starts GitLab container and waits for healthy status
4. Extracts initial root password and creates API personal access token
5. Generates SSL certificates for container registry
6. Configures GitLab registry settings and reconfigures GitLab
7. Creates demo project with sample CI/CD pipeline
8. Registers and configures GitLab Runner with Docker executor
9. Cleans up temporary access tokens

## Troubleshooting

If installation fails on DNS resolution, ensure hosts file entries exist:
- Local: `127.0.0.1 local-gitlab.me registry.local-gitlab.me`
- Remote: `IP_ADDRESS hostname.me registry.hostname.me`

The playbook will fail early with helpful DNS configuration messages if connectivity checks fail.