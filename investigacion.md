## Sí, hay imágenes Docker listas para usar

**Buena noticia**: hay varias opciones disponibles. Aquí están las principales:[1][2][3][4]

### Opción 1: **heyvaldemar/quake3-server** (Recomendado - Más mantenido)

Esta es la más profesional y activamente mantenida (actualizada en 2024): Incluye Docker Compose completo y permite configuración fácil via archivos `.env` y `server.cfg`, exactamente como probablemente hiciste con RA3.[5][1]

Simplemente clona el repositorio y ejecutas:[1]

```bash
docker compose -f quake3-server-docker-compose.yml -p quake3-server up -d
```

Después cambias `server.cfg` y reinician los contenedores. También incluye una interfaz web en el puerto 80.

### Opción 2: **wokoman/quake3-osp-server** (Simple, Instagib listo)

Esta imagen ya viene **preconfigurada con OSP e Instagib habilitado**: Más antigua (última actualización 2020) pero totalmente estable. Ideal si solo quieres FFA + Instagib sin complicaciones:[6]

```bash
docker run --name q3a-server -d -p 27960:27960/udp wokoman/quake3-osp-server
```

Sin necesidad de Docker Compose, un comando simple.

### Opción 3: **exesse/q3a-server** (Minimalista)

Basada en ioquake3, ligera y permite montar configs personalizadas:[3]

```bash
docker run --name q3a-server -d \
  -p 27960:27960/udp \
  -v /ruta/config:/home/ioq3srv/.q3a/baseq3/server.cfg \
  exesse/q3a-server:latest
```

### Para múltiples servidores (FFA + Instagib)

Con **heyvaldemar**, puedes crear un `docker-compose.yml` con dos servicios:[5][1]

```yaml
services:
  q3-ffa:
    image: heyvaldemar/quake3-server
    ports:
      - "27960:27960/udp"
    volumes:
      - ./ffa-server.cfg:/config/server.cfg

  q3-instagib:
    image: heyvaldemar/quake3-server
    ports:
      - "27961:27961/udp"
    volumes:
      - ./instagib-server.cfg:/config/server.cfg
```

Y cada uno usa su propio `server.cfg` con puertos diferentes (27960 y 27961).

### Mi recomendación

**Usa `heyvaldemar/quake3-server`** si quieres un setup profesional y mantenido como hiciste con RA3. Si prefieres algo más simple y directo, **`wokoman/quake3-osp-server`** ya viene listo con OSP e instagib sin necesidad de configurar casi nada.

¿Cuál prefieres? ¿O necesitas help montando el docker-compose.yml específico para tus dos servidores?

[1](https://www.heyvaldemar.com/install-quake3-server-using-docker-compose)
[2](https://hub.docker.com/r/heyvaldemar/quake3-server)
[3](https://exesse.github.io/quake3-arena-server/)
[4](https://hub.docker.com/r/adobromilskiy/quake3-server)
[5](https://github.com/heyvaldemar/quake3-server-docker-compose)
[6](https://hub.docker.com/r/wokoman/quake3-osp-server)
[7](https://blog.no42.org/article/ioquake3-docker/)
[8](https://hub.docker.com/r/ch0ww/q3dc)
[9](https://gcore.com/blog/install-quake-iii-arena-in-docker)
[10](https://www.reddit.com/r/selfhosted/comments/1osmqzg/quakejs_container_quake_3_arena_in_the_browser/)
[11](https://www.youtube.com/watch?v=kWL1IkPvecE)
[12](https://github.com/heyvaldemar/quake3-server-docker-compose/actions)
[13](https://www.youtube.com/watch?v=KfnzpEtDE8A)
[14](https://github.com/exesse/quake3-arena-server)
[15](https://registry.hub.docker.com/search?s=stars&architecture=arm64&operating_system=linux&categories=Operating+Systems&page=48&categories=Message+Queues)
[16](https://hub.docker.com/r/wokoman/quake3-osp-server/tags)