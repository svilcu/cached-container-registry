# Cached Container Registry

A project to set up a **Docker Proxy Cache** using **Docker Registry** on **Podman**, automated with **Ansible**. This setup enables local caching of container images, improving performance, reducing bandwidth usage, and enhancing reliability when pulling images from external registries.

## Features

- **Automated setup** with Ansible
- **Reduces bandwidth usage** by caching pulled images locally
- **Supports multiple upstream registries** including Docker Hub, Quay, GHCR, GCR, and Kubernetes registry
- **Speeds up deployments** by serving cached images

## Prerequisites

- **Ansible** (version 2.18.2 but earlier versions may work)
- **Podman** (version 3.4.4 or later)
- **Docker Registry** (running in Podman)
- **Linux-based OS** Tested on Armbian 24.11.1 Jammy (Raspberry Pi 5), but it should work on most Debian/Ubuntu-based distributions.

## Installation

As it is, Ansible will install and configure Podman on a remote machine since this playbook is part of a larger project. However, I will show how to configure it to run on the local machine.

### 1. Clone the Repository

```bash
git clone https://github.com/svilcu/cached-container-registry.git
cd cached-container-registry
```

### 2. Install Dependencies

Before proceeding, ensure **Python 3** and **make** are installed. The Makefile automates dependency installation

```bash
cd requirements; make
```

### 3. Configure Ansible Inventory

Update `inventory.yml` with your target machine's details or IP address. If you have a private DNS setup, you can use a hostname like `cache.home.arpa`, as I do.:

```yaml
---
proxy:
  hosts:
    cache.home.arpa:
```

Or, if you want to run it locally

```yaml
---
all:
  children:
    proxy:
      hosts:
        localhost:
          ansible_connection: local
```

### 4. Configure ansible.cfg

I am using a separate user **cadmin** on the remote machine that has the right to sudo. You may want to change this in ansible.cfg or use **root** if you do not use a separate user.

```ini
remote_user=cadmin
```

### 5. Run the Playbook

```bash
ansible-playbook site.yml
```

## Usage

After deployment, the registry will be accessible at `http://your_host:500X` (where X is 0-4, depending on the upstream registry configured).

The following `registry.yaml` should be placed in `/etc/rancher/k3s/` to configure **k3s** to use the cached registries

```yaml
mirrors:
  docker.io:
    endpoint:
      - "http://cache.home.arpa:5000"
  quay.io:
    endpoint:
      - "http://cache.home.arpa:5004"
  ghcr.io:
    endpoint:
      - "http://cache.home.arpa:5003"
  gcr.io:
    endpoint:
      - "http://cache.home.arpa:5002"
  registry.k8s.io:
    endpoint:
      - "http://cache.home.arpa:5001"
```

To check cached images:

```bash
curl -X GET http://your_host:500[0-4]/v2/_catalog
```

## Configuration

By default, images are cached in `/var/lib/registry`. You can modify this path in the playbook to suit your needs.

## Logs & Debugging

Containers are named based on the registry they cache: docker, k8s, ghcr, gcr, quay

To view registry logs:

```bash
podman logs -f container_name
```

## License

This project is licensed under the MIT License. See `LICENSE` for details.

## Contributions

Contributions are welcome! Feel free to submit issues or pull requests.

## Author

[Stefanita Vilcu](https://github.com/svilcu)
