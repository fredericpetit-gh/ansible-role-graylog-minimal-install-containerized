# Rôle Ansible "graylog-completed"

Rôle Ansible pour déployer Graylog avec MongoDB et OpenSearch en conteneurs.

## Description

Ce rôle lance trois conteneurs :
- MongoDB 6-jammy
- OpenSearch 1
- Graylog 6 (exposé sur le port 9009)

## Variables par défaut

| Variable                    | Description                         | Valeur par défaut                |
|-----------------------------|-------------------------------------|----------------------------------|
| `mongo_image`               | Image Docker MongoDB                | `mongo:6-jammy`                  |
| `opensearch_image`          | Image Docker OpenSearch             | `opensearchproject/opensearch:1` |
| `graylog_image`             | Image Docker Graylog                | `graylog/graylog:6`              |
| `graylog_admin_password`    | Mot de passe administrateur Graylog | `root`                           |
| `graylog_password_secret`   | Clé secrète pour Graylog            | `root`                           |
| `graylog_http_external_uri` | URL externe Graylog                 | `http://localhost:9009/`         |

## Prérequis

Ce rôle nécessite :
- Docker ou Podman installé sur l'hôte cible.
- GIT installé sur l'hôte cible.
- La collection Ansible `community.docker` installée localement (`ansible-galaxy collection install community.docker`).

## Installation

À ajouter dans le fichier requirements.yaml :

```yaml
- src: git+https://gitlab.com/fredericpetit/ansible-role-graylog-completed.git
  name: graylog-completed
  version: main
```

Puis faire :

`ansible-galaxy role install -r requirements.yaml`

## Utilisation

Ajouter le rôle dans un playbook :

```yaml
---
- name: Déployer Graylog avec MongoDB et OpenSearch (conteneurs)
  hosts: docker
  become: yes
  gather_facts: true
  vars:
    remove_container: true
  roles:
    - graylog-completed

  tasks:
    - name: Supprimer les containers Graylog, MongoDB et OpenSearch
      docker_container:
        name: "{{ item }}"
        state: absent
      loop:
        - "log-graylog"
        - "log-mongo"
        - "log-opensearch"
      when: remove_container | bool
```

puis lancer avec (par exemple) `ansible-playbook -i inventory playbooks/graylog/install.yaml`

## Notes

### Limites système (ulimits) pour OpenSearch

OpenSearch nécessite une limite élevée du nombre de fichiers ouverts (`nofile = 65536`) pour fonctionner correctement. Si cette limite n’est pas respectée, le conteneur échouera avec une erreur liée à `RLIMIT_NOFILE`.

Ce rôle configure automatiquement cette limite via le paramètre `ulimits` dans la tâche de lancement du conteneur Docker. Aucune configuration manuelle supplémentaire n’est nécessaire si vous utilisez ce rôle tel quel.

Si vous exécutez les conteneurs manuellement ou que votre hôte impose des restrictions, vous pouvez augmenter les limites système dans la configuration de votre utilisateur (`limits.conf`) ou dans la configuration du démon Docker (`daemon.json`) et redémarrer Docker.