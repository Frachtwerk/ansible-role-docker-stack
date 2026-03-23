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
          - "chat.myservice/hook/12345"
          - "smtp://user@example.com:secret@smtp.example.com:587/?fromaddress=sender@example.com\
            &toaddresses=recipient@example.com&encryption=ExplicitTLS&usestarttls=yes"
        docker_watchtower_disabled_containers_list:
          - myservice
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
docker_traefik_basic_auth: []
docker_traefik_root_url: ""
docker_traefik_certs_crt_file: ""
docker_traefik_certs_key_file: ""

###############
# watchtower  #
###############

docker_watchtower_path: "/opt/watchtower"
docker_watchtower_dynamic_user: root  # for e.g user namespace remapping
docker_watchtower_dynamic_group: root
docker_watchtower_timezone: "Europe/Berlin"
docker_watchtower_poll_interval: ""  # in seconds
docker_watchtower_schedule: "0 30 4 * * 2,4"  # Every Tuesday and Thursday at 4:30:00
docker_watchtower_enable_labels: false  # "com.centurylinklabs.watchtower.enable: true"
docker_watchtower_api_token:
docker_watchtower_enable_api: false  # to manualy trigger updates
docker_watchtower_notification_url: []
docker_watchtower_notifications_hostname: "{{ inventory_hostname }}"
docker_watchtower_notification_title_tag: "{{ inventory_hostname }}"
docker_watchtower_disabled_containers_list: []  # Regex patterns are supported
docker_watchtower_log_level: info  # panic, fatal, error, warn, info, debug, trace
docker_watchtower_log_format: Auto  # Auto, LogFmt, Pretty, JSON
docker_watchtower_rolling_restart: false

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
