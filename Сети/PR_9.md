1) docker-compose.yml
2) развертывание виртаульных маршрутизаторов


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