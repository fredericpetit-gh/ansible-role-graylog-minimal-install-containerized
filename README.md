# Ansible Role "fredericpetit-gh.graylog-minimal-install-containerized"

[🇲🇫] Rôle Ansible pour déployer Graylog avec MongoDB et OpenSearch, en conteneurs avec installation minimale. - [🇬🇧] Ansible role for deploying Graylog with MongoDB and OpenSearch, containerized with minimal installation.

**WORKS WITH DOCKER & PODMAN !**

## Description

This role starts three containers :
- MongoDB 6-jammy
- OpenSearch 1
- Graylog 6 (exposed on port 9000)

## Default Variables

| Variable                    | Description               | Default Value                                     |
|-----------------------------|---------------------------|---------------------------------------------------|
| `mongo_image`               | Image Docker MongoDB      | `mongo:6-jammy`                                   |
| `opensearch_image`          | Image Docker OpenSearch   | `opensearchproject/opensearch:1`                  |
| `graylog_image`             | Image Docker Graylog      | `graylog/graylog:6`                               |
| `mongo_container_name`      | Container name MongoDB    | `ubuntu-mongo`                                    |
| `opensearch_container_name` | Container name OpenSearch | `amazon-opensearch`                               |
| `graylog_container_name`    | Container name Graylog    | `ubuntu-graylog`                                  |
| `graylog_admin_password`    | Password Graylog          | `root`                                            |
| `graylog_password_secret`   | Secret key Graylog        | `root`                                            |
| `graylog_http_external_uri` | URL Graylog               | `http://{{ ansible_default_ipv4.address }}:9000/` |

## Requirements

This role requires :
- Docker OR Podman installed on the target host.
- The Ansible collection `community.docker` installed locally (`ansible-galaxy collection install community.docker`) for Docker OR `containers.podman`(`ansible-galaxy collection install containers.podman`) for Podman.

## Installation

To add in the requirements.yaml file :

```yaml
- src: fredericpetit-gh.graylog-minimal-install-containerized
  version: 1.0.3
```

OR

```yaml
- src: git+https://gitlab.com/fredericpetit/ansible-role-graylog-minimal-install-containerized.git
  name: graylog-minimal-install-containerized
  version: main
```

Then run : `ansible-galaxy role install -r requirements.yaml --force`

## Usage

Add the role in a playbook, like this for example :

```yaml
- name: Deploy Graylog with MongoDB and OpenSearch (containers)
  hosts: docker
  become: true
  gather_facts: true
  vars:
    remove_container: true
    graylog_ports:
      - "9009:9000"
      - "12201:12201/udp"
    container_driver: docker  # or 'podman', set externally or via group_vars

  pre_tasks:
    # Remove containers with Docker
    - name: Remove Graylog, MongoDB and OpenSearch (Docker)
      docker_container:
        name: "{{ item }}"
        state: absent
      loop:
        - "{{ graylog_container_name }}"
        - "{{ mongo_container_name }}"
        - "{{ opensearch_container_name }}"
      when:
        - remove_container | bool
        - container_driver == "docker"

    # Remove containers with Podman
    - name: Remove Graylog, MongoDB and OpenSearch (Podman)
      containers.podman.podman_container:
        name: "{{ item }}"
        state: absent
      loop:
        - "{{ graylog_container_name }}"
        - "{{ mongo_container_name }}"
        - "{{ opensearch_container_name }}"
      when:
        - remove_container | bool
        - container_driver == "podman"

  roles:
    - fredericpetit-gh.graylog-minimal-install-containerized
```

Then run : `ansible-playbook -i inventory playbooks/graylog/install.yaml`

## Notes

### [🇲🇫] Limites système (ulimits) pour OpenSearch

OpenSearch nécessite une limite élevée du nombre de fichiers ouverts (`nofile = 65536`) pour fonctionner correctement. Si cette limite n’est pas respectée, le conteneur échouera avec une erreur liée à `RLIMIT_NOFILE`.

Il est possible de configurer automatiquement cette limite via le paramètre `ulimits` dans la tâche de lancement du conteneur, ou augmenter les limites système dans la configuration de l'utilisateur (`limits.conf`) ou dans la configuration du démon (`daemon.json` pour Docker) et redémarrer.

### [🇬🇧] System limits (ulimits) for OpenSearch

OpenSearch requires a high open file limit (`nofile = 65536`) to function correctly. If this limit is not met, the container will fail with an `RLIMIT_NOFILE` error.

You can automatically set this limit using the `ulimits` parameter when launching the container, or increase system limits in the user configuration (`limits.conf`) or in the daemon configuration (`daemon.json` for Docker), then restart.