# Wireguard server ansible role

Suitable to deploy https://github.com/linuxserver/docker-wireguard

## Configuration

```yaml
wireguard_service_name: wireguard
wireguard_docker_image: lscr.io/linuxserver/wireguard

wireguard_host_config_dir: /etc/wireguard
wireguard_host_memory: 32m
wireguard_host_port: 51820

wireguard_port_mappings: []

wireguard_peers:
  - name: peer1
    allowed_ips:
  - name: peer2
    allowed_ips:

wireguard_log_confs: True
```

### Route inbound traffic to wireguard

```yaml
wireguard_port_mappings:
  - 80:8080/tcp
  - 443:8443/tcp
```

### Configure peers

```yaml
wireguard_peers:
  - name: peer1
    allowed_ips: 192.168.1.0/24
  - name: peer2
    allowed_ips: 192.168.2.0/24
  - name: peer3
    allowed_ips: 192.168.3.0/24
```

### Customize iptables rules

For example, these rules routes inbound traffic to "internal" web servers:

```yaml
wireguard_post_up: >-
  iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 10.13.13.2:8080;
  iptables -t nat -A PREROUTING -p tcp --dport 8443 -j DNAT --to-destination 10.13.13.3:8443;
  iptables -t nat -A POSTROUTING -p tcp -d 10.13.13.0/24 -j MASQUERADE;
  iptables -A FORWARD -i %%i -j ACCEPT; iptables -A FORWARD -o %%i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
wireguard_post_down: >-
  iptables -t nat -D PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 10.13.13.2:8080;
  iptables -t nat -D PREROUTING -p tcp --dport 8443 -j DNAT --to-destination 10.13.13.3:8443;
  iptables -t nat -D POSTROUTING -p tcp -d 10.13.13.0/24 -j MASQUERADE;
  iptables -D FORWARD -i %%i -j ACCEPT; iptables -D FORWARD -o %%i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE
```
