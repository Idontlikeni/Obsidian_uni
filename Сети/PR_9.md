1) docker-compose.yml
2) развертывание виртаульных маршрутизаторов
3) hosts.yml
4) шаблон jinja2
5) site.yml
6) Запуск плейбука![[Pasted image 20261111111946.png]]
7) проверка OSPF-средства![[Pasted image 20261111112018.png]]
8) napalm_verify.py
9) верификация состояния![[Pasted image 20261111112525.png]]
10) docker-compose-plugin
11) backup_configs.sh
12) 

docker-compose.yml:
```yml
version: "3.8"
networks:
  prac_net:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br-prac-net
      com.docker.network.bridge.enable_icc: "true"
    ipam:
      config:
        - subnet: 172.30.0.0/24

services:
  r1:
    image: frrouting/frr:latest
    container_name: frr-r1
    cap_add: [NET_ADMIN, NET_RAW, SYS_ADMIN]
    volumes: ["./configs/r1:/etc/frr"]
    networks:
      prac_net: { ipv4_address: 172.30.0.11 }
    command: ["/bin/sh", "-c", "apk add python3 && /usr/lib/frr/docker-start"]

  r2:
    image: frrouting/frr:latest
    container_name: frr-r2
    cap_add: [NET_ADMIN, NET_RAW, SYS_ADMIN]
    volumes: ["./configs/r2:/etc/frr"]
    networks:
      prac_net: { ipv4_address: 172.30.0.12 }
    command: ["/bin/sh", "-c", "apk add python3 && /usr/lib/frr/docker-start"]

  r3:
    image: frrouting/frr:latest
    container_name: frr-r3
    cap_add: [NET_ADMIN, NET_RAW, SYS_ADMIN]
    volumes: ["./configs/r3:/etc/frr"]
    networks:
      prac_net: { ipv4_address: 172.30.0.13 }
    command: ["/bin/sh", "-c", "apk add python3 && /usr/lib/frr/docker-start"]
```

Шаблон jinja2:
```j2
hostname {{ inventory_hostname }}
!
router ospf
 ospf router-id {{ router_id }}
{% for net in ospf_networks %}
 network {{ net }} area {{ ospf_area }}
{% endfor %}
!
line vty
!
```

hosts.yml:
```yml
all:
  hosts:
    r1:
      ansible_host: frr-r1
      ansible_connection: docker
      router_id: 1.1.1.1
      ospf_area: 0.0.0.0
      ospf_networks:
        - 172.30.0.11/32
    r2:
      ansible_host: frr-r2
      ansible_connection: docker
      router_id: 2.2.2.2
      ospf_area: 0.0.0.0
      ospf_networks:
        - 172.30.0.12/32
    r3:
      ansible_host: frr-r3
      ansible_connection: docker
      router_id: 3.3.3.3
      ospf_area: 0.0.0.0
      ospf_networks:
        - 172.30.0.13/32
```

site.yml:
```yml
- name: Применение конфигурации FRR (Network as Code)
  hosts: all
  gather_facts: no
  tasks:
    - name: Сгенерировать frr.conf из шаблона
      template:
        src: templates/frr.conf.j2
        dest: /etc/frr/frr.conf
      notify: Перезагрузить конфигурацию FRR

  handlers:
    - name: Перезагрузить конфигурацию FRR
      command: vtysh -b
```

napalm_verify.py
```yml
#!/usr/bin/env python3
import json
import subprocess

def get_facts():
    cmd_version = "sudo docker exec frr-r1 vtysh -c 'show version'"
    res = subprocess.check_output(cmd_version, shell=True, text=True, stderr=subprocess.DEVNULL)
    version_line = [line for line in res.split('\n') if "FRR" in line or "FRRouting" in line]
    return {
        "hostname": "frr-r1",
        "vendor": "FRRouting",
        "model": "Docker Container",
        "os_version": version_line[0].strip() if version_line else "Unknown FRR Version",
    }

if __name__ == "__main__":
    print("--- [NAPALM Verification emulation] get_facts() ---")
    print(json.dumps(get_facts(), indent=4, ensure_ascii=False))
```

```sh
#!/bin/bash
BACKUP_DIR=~/netmon/frr_lab/backups
mkdir -p "$BACKUP_DIR"
TS=$(date +%Y%m%d_%H%M%S)
for r in r1 r2 r3; do
  sudo docker exec frr-$r cat /etc/frr/frr.conf > "$BACKUP_DIR/${r}_${TS}.conf"
done
echo "[$(date +%T)] Резервная копия создана: $TS"
```