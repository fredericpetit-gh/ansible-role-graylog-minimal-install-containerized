# Rôle Ansible "graylog-minimal-install-containerized"

[🇲🇫] Rôle Ansible pour déployer Graylog avec MongoDB et OpenSearch, en conteneurs avec installation minimale. - [🇬🇧] Ansible role for deploying Graylog with MongoDB and OpenSearch, containerized with minimal installation.

## Description

Ce rôle lance trois conteneurs :
- MongoDB 6-jammy
- OpenSearch 1
- Graylog 6 (exposé sur le port 9009)

## Variables par défaut

| Variable                    | Description                         | Valeur par défaut                                 |
|-----------------------------|-------------------------------------|---------------------------------------------------|
| `mongo_image`               | Image Docker MongoDB                | `mongo:6-jammy`                                   |
| `opensearch_image`          | Image Docker OpenSearch             | `opensearchproject/opensearch:1`                  |
| `graylog_image`             | Image Docker Graylog                | `graylog/graylog:6`                               |
| `graylog_admin_password`    | Mot de passe administrateur Graylog | `root`                                            |
| `graylog_password_secret`   | Clé secrète pour Graylog            | `root`                                            |
| `graylog_http_external_uri` | URL externe Graylog                 | `http://{{ ansible_default_ipv4.address }}:9009/` |

## Prérequis

Ce rôle nécessite :
- Docker ou Podman installé sur l'hôte cible.
- GIT installé sur l'hôte cible.
- La collection Ansible `community.docker` installée localement (`ansible-galaxy collection install community.docker`).

## Installation

À ajouter dans le fichier requirements.yaml :

```yaml
- src: git+https://gitlab.com/fredericpetit/ansible-role-graylog-minimal-install-containerized.git
  name: graylog-minimal-install-containerized
  version: main
```

Puis faire :

`ansible-galaxy role install -r requirements.yaml --force`

## Utilisation

Ajouter le rôle dans un playbook :

```yaml
- name: Déployer Graylog avec MongoDB et OpenSearch (conteneurs)
  hosts: docker
  become: yes
  gather_facts: true
  vars:
    remove_container: true

  pre_tasks:
    - name: Supprimer Graylog, MongoDB et OpenSearch
      docker_container:
        name: "{{ item }}"
        state: absent
      loop:
        - "{{ graylog_container_name }}"
        - "{{ mongo_container_name }}"
        - "{{ opensearch_container_name }}"
      when: remove_container | bool

  roles:
    - graylog-minimal-install-containerized
```

puis lancer avec (par exemple) `ansible-playbook -i inventory playbooks/graylog/install.yaml`

## Notes

### Limites système (ulimits) pour OpenSearch

OpenSearch nécessite une limite élevée du nombre de fichiers ouverts (`nofile = 65536`) pour fonctionner correctement. Si cette limite n’est pas respectée, le conteneur échouera avec une erreur liée à `RLIMIT_NOFILE`.

Il est possible de configurer automatiquement cette limite via le paramètre `ulimits` dans la tâche de lancement du conteneur, ou augmenter les limites système dans la configuration de l'utilisateur (`limits.conf`) ou dans la configuration du démon (`daemon.json` pour Docker) et redémarrer.