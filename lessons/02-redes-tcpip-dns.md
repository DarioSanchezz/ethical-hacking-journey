# Lección 02: Redes (TCP/IP, puertos, DNS y HTTP)

## Salida de `ip a`
| Campo | Significado |
|---|---|
| `lo` | Loopback, la máquina hablando consigo misma (127.0.0.1) |
| `eth0` | La tarjeta de red virtual |
| `state UP` | Interfaz activa |
| `inet 10.0.2.15/24` | Mi IP y la máscara (/24 = 254 equipos, rango .1 a .254) |
| `link/ether` | Dirección MAC (los 3 primeros bloques identifican al fabricante) |
| `brd` | Dirección de broadcast |
| `inet6` | IPv6 |

Otros comandos: `ip -br a` (resumen), `ip route` (puerta de enlace, línea `default via`).

## `ping -c 4 8.8.8.8`
- `-c 4`: enviar 4 paquetes y parar (en Linux, sin `-c`, ping no termina nunca).
- `ttl`: se resta 1 en cada router; valores iniciales típicos: 64 Linux, 128 Windows, 255 equipos de red.
- Ping a IP funciona y a nombre no → problema de **DNS**.

## Modelo TCP/IP
| Capa | Qué hace | Ejemplos |
|---|---|---|
| Aplicación | El programa que habla | HTTP, DNS, SSH |
| Transporte | Entrega entre programas | TCP, UDP |
| Internet | Lleva el paquete entre redes | IP, ICMP |
| Acceso a red | Envío físico | Ethernet, Wi-Fi (MAC) |

## TCP frente a UDP
TCP abre conexión y garantiza entrega; UDP envía sin garantías pero más rápido.

### Saludo de tres vías
1. SYN → 2. SYN-ACK → 3. ACK.
Es la base del escaneo de puertos: SYN-ACK = puerto abierto, RST = cerrado, sin respuesta = filtrado.

## Puertos importantes
| Puerto | Servicio |
|---|---|
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 139/445 | SMB |
| 443 | HTTPS |
| 3306 | MySQL |
| 3389 | RDP |

## DNS
Traduce nombres a IPs. Registros: A, AAAA, MX, NS, TXT, CNAME.
`dig google.com`, `dig google.com MX`, `dig +short google.com`.

## HTTP
Petición (método + ruta + cabeceras) y respuesta (código + cabeceras + cuerpo).
Códigos: 200 OK, 301/302 redirección, 401, 403, 404, 500.
`curl -I URL` muestra solo las cabeceras; `curl -v URL` muestra todo el intercambio.

## Comandos de la lección
- `sudo ss -tulpn`: puertos en escucha (t=TCP, u=UDP, l=listening, p=proceso, n=numérico).
- `sudo systemctl start ssh` / `stop ssh`: arrancar/parar un servicio.
- `traceroute 8.8.8.8`: ruta de un paquete.

## Qué he aprendido
Aprendí a leer la salida de `ip a`: la IP y la máscara me dicen qué rango de red
tengo delante, que es justo lo que luego escaneo. Entendí por qué `ping` necesita `-c`
en Linux y cómo diagnosticar con dos pings si el fallo está en la conexión o en el DNS.
Vi la diferencia entre TCP y UDP y, sobre todo, el saludo de tres vías (SYN, SYN-ACK,
ACK), porque es la base de cómo nmap decide si un puerto está abierto, cerrado o
filtrado. Me aprendí los puertos más comunes y por qué cada uno interesa a un atacante.
Con `ss -tulpn` comprobé que un puerto abierto es simplemente un servicio escuchando,
con `dig` consulté registros DNS y con `curl -I` miré cabeceras HTTP, que a veces
revelan qué servidor y versión corre una web.
