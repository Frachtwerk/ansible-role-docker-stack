# fw_oss.docker_stack

Set up [portainer](https://docs.portainer.io/), [traefik](https://doc.traefik.io/traefik/) and [watchtower](https://containrrr.dev/watchtower/) in docker

## Requirements

Docker up and running

## Dependencies

[Community General Collection](https://docs.ansible.com/ansible/latest/collections/community/general/index.html) (comes with `ansible`, but not with `ansible-core`)

## Usage

### Traefik

#### DNS Challenge

Activate by setting  `docker_traefik_dns_challenge: true`  
Requires:

- Provider: `docker_traefik_dns_provider`
- Resolvers (sometimes): `docker_traefik_dns_resolver`

Anything further is communicated via the env of the traefik container, e.g.:  

```yaml
docker_traefik_custom_environment:
  CLOUDFLARE_DNS_API_TOKEN: "1234567890abcdefghijklmnopqrstuvwxyz"
```

See the [lego docs](https://go-acme.github.io/lego/dns/index.html) for your provider

## Example Playbook

Depending on what loadout you wanna achieve:

```yaml
    - name: Install Stack on Docker host
    - hosts: docker
      become: true
      vars:
        docker_install_traefik: true
        docker_install_watchtower: true
        docker_install_portainer: true
        docker_install_portainer_agent: true
        docker_portainer_healthcheck:
          test: "wget --no-verbose --no-check-certificate --tries=1 --spider https://localhost:9443 || exit 1"
          interval: 30s
          timeout: 5s
          retries: 3
          start_period: 20s
      roles:
        - ansible_role_docker_stack
```

```yaml
    - name: Install traefik with wildcard certificates on Docker host
    - hosts: docker
      become: true
      vars:
        docker_install_traefik: true
        docker_traefik_https_enabled: true
        docker_traefik_enable_acme: true
        docker_traefik_wildcard_list: ["example.com","internal.example.com"]
        docker_traefik_enable_stored_certs: false
        docker_traefik_acme_mail: "admin@example.com"
        docker_traefik_dns_challenge: true
        docker_traefik_dns_provider: "desec"  # https://go-acme.github.io/lego/dns/
        # docker_traefik_dns_token: "abCdeFGhjk"  # obsolete
        docker_traefik_dns_resolvers: ['21.43.78.9','11.12.23.45']
        docker_traefik_custom_environment:
          'DESEC_TOKEN': "abCdeFGhjk"
      roles:
        - ansible_role_docker_stack
```

```yaml
    - name: Install traefik with stored certificates on Docker host
    - hosts: docker
      become: true
      vars:
        docker_install_traefik: true
        docker_traefik_https_enabled: true
        docker_traefik_enable_acme: false
        docker_traefik_enable_stored_certs: true
      roles:
        - ansible_role_docker_stack
```

```yaml
    - name: Install watchtower on docker
    - hosts: all
      become: true
      vars:
        docker_install_watchtower: true
        docker_watchtower_timezone: "Europe/Berlin"
        docker_watchtower_poll_interval: "7200"  # every 2h
        docker_watchtower_notification_url:
          - generic://chat.myservice.io/hook/12345?template=json&ContentType=application/json&messagekey=text&titlekey=username
          - smtp://user@example.com:secret@smtp.example.com:587/?fromaddress=sender@example.com&toaddresses=recipient@example.com&encryption=ExplicitTLS&usestarttls=yes
        docker_watchtower_disabled_containers_list:
          - myservice
        docker_watchtower_cooldown_delay: 0  # disable security cooldown
      roles:
        - ansible_role_docker_stack
```

## Role Variables

All variables and their defaults are documented in [`defaults/main.yml`](defaults/main.yml).
Variables that were renamed or removed are listed in [`vars/main.yml`](vars/main.yml) (`_docker_stack_obsolete_vars`); the role fails if one of them is still set.

Structured variables:

```yaml
docker_traefik_additional_entrypoints:
  - name: ssh
    port: 2222
    protocol: tcp  # tcp (default) or udp
  - name: alt-https
    port: 8443
    http: true  # optional, adds TLS and middlewares below
    ssl: true
    headers: true
    noindex: false
    compression: false

docker_traefik_non_docker_services:
  - name: nas
    routes:
      - url: nas.example.com
        middlewares: ["default-ipwhitelist@file"]
    servers:
      - http://192.168.1.10:5000

docker_logins:
  - docker_registry_url: registry.example.com
    docker_registry_user: user
    docker_registry_pass: secret
```

## License

MIT

## Author Information

FW-OSS, 2024
