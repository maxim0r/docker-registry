docker-registry
=========

Docker registry v2 install

Requirements
------------

Docker installed

Role Variables
--------------
```
docker_registry:
  restart: always
  image: registry:2
  port: 5000
  htpasswd: /tmp/htpasswd
  tls:
    certificate: /tmp/cert.pem
    key: /tmp/key.pem
  environment:
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
    REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    data: /var/local/registry/data
    certs: /var/local/registry/certs
    auth: /var/local/registry/auth
```


Dependencies
------------



Example Playbook
----------------

```
- name: Install Docker Registry
  hosts: docker-registry
  become: yes

  vars_files:
    - vars/credentials.yml

  pre_tasks:

    - name: create certificate
      local_action: >
        shell openssl req -batch -newkey rsa:4096 -nodes -sha256  -x509 -days 30
        -subj '/C=RU/ST=Moscow/L=Moscow/O=Example Inc/OU=Example team'
        -out ./files/registry.crt
        -keyout ./files/registry.key
      args:
        creates: files/registry.key

    - name: pull registry image
      local_action: docker_image
      args:
        name: registry
        tag: 2

    - name: create htpasswd
      local_action: >
        shell docker run --entrypoint htpasswd registry:2 -Bbn {{ infra.registry.user }} {{ infra.registry.password }} > files/htpasswd
      args:
        creates: files/htpasswd

  roles:
    - role: docker-registry
      docker_registry:
        htpasswd: /tmp/htpasswd
        tls:
          certificate: files/registry.crt
          key: files/registry.key
        port: 5000
```
License
-------

GPLv3
