# Lección 07: Entorno moderno con Docker y OWASP Juice Shop

> A partir de aquí, objetivos con tecnología actual. Juice Shop es una app de
> entrenamiento oficial de OWASP, legal para practicar.

## Por qué Docker en vez de otra VM
Un contenedor empaqueta solo la app y sus dependencias, comparte el kernel con el
anfitrión y arranca en segundos, frente a minutos y varios GB de una VM.

## Instalar Docker en Kali
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
docker --version
sudo docker info
```
Para usarlo sin `sudo` (requiere cerrar sesión y volver a entrar):
```bash
sudo usermod -aG docker $USER
```

## Arrancar Juice Shop
```bash
docker run -d -p 3000:3000 --name juiceshop bkimminich/juice-shop
```
`-d` segundo plano · `-p host:contenedor` mapea puertos · `--name` nombre del
contenedor · última parte, la imagen (`autor/nombre`) descargada de Docker Hub.

## Comprobar y acceder
```bash
docker ps
```
Navegador: `http://localhost:3000`

## Comandos básicos de Docker
| Quiero... | Comando |
|---|---|
| Ver activos / todos | `docker ps` / `docker ps -a` |
| Parar / arrancar | `docker stop nombre` / `docker start nombre` |
| Eliminar | `docker rm nombre` |
| Ver logs | `docker logs nombre` |
| Entrar dentro | `docker exec -it nombre sh` |
| Ver imágenes descargadas | `docker images` |

## Qué cambia frente a Metasploitable
No hay puertos de red con servicios vulnerables: todo el ataque ocurre **dentro de la
aplicación web** (formularios, APIs, tokens, lógica de negocio). Nmap aquí aporta poco;
lo importante empieza al explorar la propia web con las herramientas de desarrollador
del navegador (pestaña Red/Network, peticiones a `/rest/` o `/api/`).

## Qué he aprendido
Entendí la diferencia práctica entre una máquina virtual y un contenedor Docker, y por
qué para apps web modernas el contenedor es mucho más rápido y ligero. Aprendí los
comandos básicos de Docker (`run`, `ps`, `stop`, `start`, `logs`, `exec`) y levanté mi
primer objetivo con tecnología actual, Juice Shop. También entendí que el enfoque de
ataque cambia por completo frente a Metasploitable: ya no busco puertos y servicios de
red, sino que exploro la aplicación web como lo haría un usuario y reviso las peticiones
que hace al servidor, que es el primer paso de cualquier auditoría web real.
