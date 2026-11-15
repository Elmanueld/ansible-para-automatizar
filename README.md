# Laboratorio Ansible con cinco servidores Ubuntu en Docker

La práctica crea cinco contenedores Ubuntu con SSH, el usuario `ansible` (clave `ansible`) y permisos sudo sin contraseña. Ansible se ejecuta desde WSL y administra los contenedores por los puertos publicados en `localhost`.

> Las credenciales simples y la autenticación SSH por contraseña son solo para este laboratorio local. No uses esta configuración en servidores expuestos a Internet.

## Requisitos

- Docker Desktop con el motor Linux iniciado y la integración de WSL habilitada para tu distribución.
- Una distribución Ubuntu en WSL.

En PowerShell, instala WSL si todavía no está instalado y reinicia Windows si el sistema lo solicita:

```powershell
wsl --install -d Ubuntu
```

## 1. Crear y levantar los cinco servidores

Desde PowerShell o una terminal ubicada en esta carpeta:

```powershell
docker compose up --build -d
docker compose ps
```

Cada servicio usa la misma imagen construida desde `Dockerfile`. Los puertos del host `2221` a `2225` llegan al puerto SSH `22` de cada contenedor.

## 2. Instalar Ansible en Ubuntu/WSL

Abre Ubuntu (WSL) y ejecuta:

```bash
sudo apt update
sudo apt install -y ansible sshpass
```

`sshpass` permite que el cliente SSH de Ansible use la contraseña de laboratorio guardada en `inventario.ini`.

## 3. Ejecutar Ansible

Desde WSL, cambia a la carpeta del proyecto. Si está en el disco C:, por ejemplo:

```bash
cd /mnt/c/Users/hp/OneDrive/Documentos/ChatGPT/Docker/ansible-docker-lab
```

Comprueba primero la conexión a los cinco hosts:

```bash
export ANSIBLE_CONFIG="$PWD/ansible.cfg"
ansible servidores -m ping
```

Ejecuta el playbook:

```bash
ansible-playbook playbook.yml
```

El playbook actualiza APT, crea el usuario `itla`, crea `/home/ansible/app/hola.txt` e instala `cowsay` y `htop` en cada servidor.

### Alternativa: ejecutar Ansible desde un contenedor de control

Si no tienes WSL disponible, ejecuta un contenedor temporal en la red Docker del laboratorio. Desde esta carpeta, en PowerShell:

```powershell
docker run --rm --network ansible-laboratorio -v "${PWD}:/lab" -w /lab ubuntu:24.04 bash -lc "apt-get update && apt-get install -y ansible sshpass && ansible-playbook -i inventario.docker.ini playbook.yml"
```

Este comando instala Ansible solo en el contenedor temporal. `inventario.docker.ini` usa los nombres DNS de los cinco servicios dentro de la red Docker; `inventario.ini` continúa siendo el inventario para WSL mediante los puertos publicados.

## Archivos

- `Dockerfile`: imagen Ubuntu con SSH, Python 3 y usuario sudo `ansible`.
- `docker-compose.yml`: cinco servidores con puertos SSH individuales.
- `inventario.ini`: hosts, puertos y credenciales del laboratorio.
- `ansible.cfg`: inventario predeterminado y desactivación de la comprobación de claves SSH.
- `playbook.yml`: automatización solicitada.

Para detener y eliminar los contenedores:

```bash
docker compose down
```
