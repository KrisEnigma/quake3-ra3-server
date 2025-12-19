# Levantar Rocket Arena 3 en el VPS

Guía corta para levantar el server en el VPS usando la imagen oficial de RA3.

## Requisitos

- VPS Ubuntu con Docker + Docker Compose
- Emulación i386 habilitada (binfmt/qemu) para correr `linux/386` en ARM64

## Archivos locales necesarios

- `pak0.pk3`
- `ra3-server.cfg`
- `arena.cfg`
- `osp-ffa.cfg` (solo OSP)
- `osp-instagib.cfg` (solo OSP)
- `docker-compose.yml`

La imagen `ra3se/q3ra3-server:latest` ya incluye el mod y los mapas de Rocket Arena 3.

## Verificar dependencias

```bash
# Binarios requeridos
command -v docker
docker compose version
command -v git
command -v git-lfs
command -v qemu-i386-static

# Paquetes requeridos (Ubuntu)
dpkg -s docker-ce docker-ce-cli containerd.io docker-compose-plugin git git-lfs qemu-user-static >/dev/null

# Binfmt registrado para 386 (sin descargar imágenes)
if [ -f /proc/sys/fs/binfmt_misc/qemu-i386 ]; then
	head -n 1 /proc/sys/fs/binfmt_misc/qemu-i386
else
	echo "binfmt_misc qemu-i386 no registrado" >&2
fi
```

## Instalar dependencias

Si falta algo en la verificación anterior, pega estas líneas en una sesión SSH (Ubuntu):

```bash
# Solo lo necesario (ARM64 + linux/386 + LFS)

needs_apt=0
for bin in docker git-lfs qemu-i386-static; do
	if ! command -v "$bin" >/dev/null 2>&1; then
		needs_apt=1
		break
	fi
done

if [ "$needs_apt" -eq 1 ]; then
	sudo apt update
fi

# Docker + Compose (solo si faltan)
if ! command -v docker >/dev/null 2>&1; then
	sudo apt install -y ca-certificates curl gnupg lsb-release
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
	echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	sudo apt update
	sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
fi

# Git LFS (pak0.pk3 en LFS)
if ! command -v git-lfs >/dev/null 2>&1; then
	sudo apt install -y git-lfs
	git lfs install --system
fi

# QEMU i386 (solo si falta)
if ! command -v qemu-i386-static >/dev/null 2>&1; then
	sudo apt install -y qemu-user-static
fi

# Binfmt para 386 (solo si no está registrado)
if [ ! -f /proc/sys/fs/binfmt_misc/qemu-i386 ]; then
	sudo docker run --privileged --rm tonistiigi/binfmt --install 386
fi
```


## Comandos SSH

Copia y pega todo el bloque siguiente en la terminal ya conectada por SSH (desde el usuario que vaya a administrar el servidor):

```bash
# Primera vez (repo nuevo)
# Crear carpeta del repo y entrar
mkdir -p ~/q3-servers
cd ~/q3-servers

# Clonar repo con sparse checkout (solo archivos necesarios)
git clone --filter=blob:none --sparse https://github.com/KrisEnigma/quake3-ra3-server.git .
git sparse-checkout init --no-cone
git sparse-checkout set /docker-compose.yml /ra3-server.cfg /arena.cfg /osp-ffa.cfg /osp-instagib.cfg /pak0.pk3

# Bajar objetos LFS (pak0.pk3 si está en LFS)
git lfs pull --include="pak0.pk3" || true

# Levantar contenedores
sudo docker compose up -d
```

## Actualizar (repo ya clonado)

```bash
cd ~/q3-servers
git sparse-checkout init --no-cone
git sparse-checkout set /docker-compose.yml /ra3-server.cfg /arena.cfg /osp-ffa.cfg /osp-instagib.cfg
git pull --ff-only
sudo docker compose restart
```

## Verificar

```bash
cd ~/q3-servers && sudo docker compose ps && sudo docker compose logs --tail=120
```

Logs esperados:
- `VM_LoadDLL 'arena/qagamei386.so' ok`
- `gamename: RA3 1.76`
- `>>> RA3 SERVER CONFIG LOADED <<<`

## Reiniciar si cambias config

```bash
cd ~/q3-servers && sudo docker compose restart
```

## Actualizar archivos y reiniciar

```bash
cd ~/q3-servers && git pull && git lfs pull && sudo docker compose restart
```

## docker-compose vs ra3-server.cfg

En `docker-compose.yml` van cvars de arranque:
- `fs_game`, `net_port`, `vm_game`, `sv_pure`, `bot_enable`, `dedicated`
- `com_hunkmegs` (solo se lee al inicio)

En `ra3-server.cfg` van cvars de gameplay y servidor:
- `sv_hostname`, `g_motd`, `sv_fps`, `sv_maxclients`, `sv_maxRate`, etc.
