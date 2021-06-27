Ansible Role: Jellyfin for Kubernetes
=====================================

Ansible role to install Jellyfin on Kubernetes.

It uses Statefulset to avoid simultaneous locks on the database.

Role Variables
--------------

```yaml
# Namespace
kubernetes_jellyfin_namespace: "default"
# App name (used as selector)
kubernetes_jellyfin_app: "jellyfin"
# Statefulset name
kubernetes_jellyfin_statefulset: "jellyfin"
# Service name
kubernetes_jellyfin_service: "jellyfin"

# Number of replicas
kubernetes_jellyfin_replicas: 1
kubernetes_jellyfin_revision_history: 1

# Node selector
kubernetes_jellyfin_node_selector: {}

# Add custom labels in the statefulset metadata section
kubernetes_jellyfin_statefulset_labels: {}
# Add custom annotations in the statefulset metadata section
kubernetes_jellyfin_statefulset_annotations: {}

kubernetes_jellyfin_resources:
  limits:
    memory: "1.5Gi"
  requests:
    memory: "512Mi"

# Setup ingress for jellyfin
kubernetes_jellyfin_setup_ingress: false
kubernetes_jellyfin_ingress:
  name: "jellyfin-ingress"
  host: "jellyfin.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

# Downloads volumes. Mounted in /media/ (see examples for more details)
kubernetes_jellyfin_media_volumes: {}

# Enby config volume. Contains the database and config.
kubernetes_jellyfin_config_volume:
  definition:

# Enby config volume. Contains the arts and cache.
kubernetes_jellyfin_cache_volume:
  definition:
```

Dependencies
------------

Kubectl needs to be installed on the host targeted by the role.


Example Playbook
----------------

```yaml

- hosts: kube-master
  run_once: true
  vars:
    kubernetes_jellyfin_media_volumes:
      # This volume will be mounted as /media/tv
      tv:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-videos
            readOnly: false
        subPath: "tv"
      movies:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-videos
            readOnly: false
        subPath: "movies"
      music:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-music
            readOnly: false

    kubernetes_jellyfin_config_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-jellyfin
          readOnly: false
        subPath: "config"

    kubernetes_jellyfin_cache_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-jellyfin
          readOnly: false
        subPath: "cache"

    kubernetes_jellyfin_setup_ingress: true
    kubernetes_jellyfin_ingress:
      name: "jellyfin-ingress"
      host: "jellyfin.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "jellyfin-ingress-tls"
          hosts:
            - "jellyfin.example.com"
  roles:
    - role: aruhier.kubernetes_jellyfin
```

Use `run_once` to run the role on only one available master in the cluster.

If the jellyfin web interface is not reachable, please check that it is listening
on `0.0.0.0`, and not only `localhost`. Also check that the public http port is
setup as 8096.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
