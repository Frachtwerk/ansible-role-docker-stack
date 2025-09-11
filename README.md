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
        watchtower_schedule: "0 0 3 1,3 * *"
        watchtower_notification_service: "shoutrrr"
        watchtower_notification_url: "chat.myservice/hook/12345"
      roles:
        - ansible_role_docker_stack
```

## Role Variables

```yml

docker_install_traefik: false
docker_install_watchtower: false
docker_install_portainer: false
docker_install_portainer_agent: false

##############
# Portainer  #
##############

docker_portainer_image: portainer/portainer-ce
docker_portainer_version: "latest"
docker_portainer_parameter: "" # e.g.--logo
docker_portainer_root_url: ""
docker_portainer_parameter:

############
# Traefik  #
############

docker_traefik_version: "v2.11"
docker_traefik_path: "/opt/traefik/"
docker_traefik_network_name: "proxy"
docker_traefik_network_enable_ipv6: false
docker_traefik_entrypoint_name_http: "web"
docker_traefik_entrypoint_name_https: "websecure"
docker_traefik_enable_stored_certs: false
docker_traefik_enable_acme: false
docker_traefik_enable_headers: false
docker_traefik_enable_compression: false
docker_traefik_enable_noindex: false
docker_traefik_ports: []
docker_traefik_additional_entrypoints: [{
  name: ""
  port: ""
  protocol: ""
}]
docker_traefik_default_ipallowlist: []
docker_traefik_non_docker_services: [{
  name: ""
  routs: [{
    url: ""
    middlewares: ""
  }]
  servers: []
  traefik_default_networks: []
}]
docker_traefik_trusted_proxies: []
docker_traefik_https_enabled: true
docker_traefik_dashboard_enable: false
docker_traefik_metrics_external: false
docker_traefik_root_url: "{{ inventory_hostname }}"
docker_traefik_dynamic_user: root
docker_traefik_dynamic_group: root
docker_traefik_force_restart: false
docker_traefik_wildcard_list: []
docker_traefik_dns_challenge: false
docker_traefik_dns_provider: ""
docker_traefik_dns_resolvers: []
docker_traefik_dns_delay: "20"
docker_traefik_default_ipwhitelist: []
docker_traefik_dashboard_basic_auth: []
docker_traefik_root_url: ""
docker_traefik_certs_crt_file: ""
docker_traefik_certs_key_file: ""

###############
# watchtower  #
###############

watchtower_poll_interval: "3600"
watchtower_schedule: "0 0 22 * * *"
watchtower_notification_service: "shoutrrr"
watchtower_notification_url: ""
watchtower_notification_service: [email, shoutrrr]
watchtower_notification_email_from: ""
watchtower_notification_email_to: ""
watchtower_notification_email_server: ""
watchtower_notification_email_server_port: ""
watchtower_notification_email_server_user: ""
watchtower_notification_email_server_password: ""
watchtower_notification_email_delay: ""
watchtower_notification_url: ""

############
# Metrics  #
############

docker_traefik_metrics: false
docker_traefik_metrics_port: number 
docker_traefik_metrics_external: false #external erreichbar
docker_traefik_metrics_network: "proxy"


###########
# Logins  #
###########

docker_logins: [{
  docker_registry_url: ""
  docker_registry_user: ""
  docker_registry_pass: ""
}]

```

## Todos

- [ ] toggle docker_traefik_force_restart

## License

MIT

## Author Information

FW-OSS, 2024
