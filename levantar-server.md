# Levantar Rocket Arena 3 en el VPS

Guía corta para levantar el server en el VPS usando la imagen oficial de RA3.

## Requisitos

- VPS Ubuntu con Docker + Docker Compose
- Emulación i386 habilitada (binfmt/qemu) para correr `linux/386` en ARM64

## Archivos locales necesarios

- `pak0.pk3`
- `server.cfg`
- `arena.cfg`
- `docker-compose.yml`

La imagen `ra3se/q3ra3-server:latest` ya incluye el mod y los mapas de Rocket Arena 3.

## Pasos (local)

```bash
# Crear carpeta remota y subir archivos
ssh ubuntu@vps.krisenigma.com "mkdir -p ~/ra3-server"
scp pak0.pk3 server.cfg arena.cfg docker-compose.yml ubuntu@vps.krisenigma.com:~/ra3-server/

# Levantar el servidor
ssh ubuntu@vps.krisenigma.com "cd ~/ra3-server && sudo docker compose up -d"
```

## Verificar

```bash
ssh ubuntu@vps.krisenigma.com "cd ~/ra3-server && sudo docker compose ps && sudo docker compose logs --tail=120"
```

Logs esperados:
- `VM_LoadDLL 'arena/qagamei386.so' ok`
- `gamename: RA3 1.76`
- `>>> RA3 SERVER CONFIG LOADED <<<`

## Reiniciar si cambias config

```bash
ssh ubuntu@vps.krisenigma.com "cd ~/ra3-server && sudo docker compose restart"
```

## Actualizar archivos y reiniciar

```bash
scp server.cfg arena.cfg docker-compose.yml ubuntu@vps.krisenigma.com:~/ra3-server/ && \
ssh ubuntu@vps.krisenigma.com "cd ~/ra3-server && sudo docker compose restart"
```

## Qué va en `docker-compose.yml` vs `server.cfg`

En `docker-compose.yml` van cvars de arranque:
- `fs_game`, `net_port`, `vm_game`, `sv_pure`, `bot_enable`, `dedicated`
- `com_hunkmegs` (solo se lee al inicio)

En `server.cfg` van cvars de gameplay y servidor:
- `sv_hostname`, `g_motd`, `sv_fps`, `sv_maxclients`, `sv_maxRate`, etc.
