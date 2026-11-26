# Ethical Hacking Journey

Mi camino de aprendizaje en hacking ético y ciberseguridad (ofensiva y defensiva).
Estudiante de Ingeniería Informática (UAX). Todo lo practico en un laboratorio
propio y aislado: Kali Linux + máquinas vulnerables en VirtualBox.

**Objetivo:** llegar a ser profesional del hacking ético, sabiendo encontrar,
explotar, documentar y también detectar y corregir vulnerabilidades.

## Laboratorio
Kali Linux y Metasploitable 2 conectados a una Red NAT de VirtualBox, aislada de mi
red doméstica. Detalles en [lab/setup.md](lab/setup.md).

## Progreso
Una lección al día.

| # | Lección | Fecha | Estado |
|---|---------|-------|--------|
| 01 | [Bases, mentalidad y terminal](lessons/01-bases-y-terminal.md) | 16/11/2026 | Hecha |
| 02 | [Redes: TCP/IP, puertos, DNS y HTTP](lessons/02-redes-tcpip-dns.md) | 16/11/2026 | Hecha |
| 03 | [Reconocimiento con nmap](lessons/03-nmap.md) | 17/11/2026 | Hecha |
| 04 | [Enumeración y búsqueda de vulnerabilidades](lessons/04-enumeracion-y-vulnerabilidades.md) | 18/11/2026 | Hecha |

| 05 | [Primera explotación con Metasploit](lessons/05-metasploit-explotacion.md) | 19/11/2026 | Hecha |
| 06 | [Cracking de contraseñas](lessons/06-cracking-contraseñas.md) | 20/11/2026 | Hecha |
| 07 | [Docker y OWASP Juice Shop](lessons/07-docker-juice-shop.md) | 23/11/2026 | Hecha |
| 08 | [Burp Suite y primeras vulnerabilidades web](lessons/08-burp-vulnerabilidades-web.md) | 23/11/2026 | Hecha |
| 09 | [XSS y manipulación de JWT](lessons/09-xss-jwt.md) | 24/11/2026 | Hecha |
| 10 | [Robo de cookies vía XSS y comparación de puntos de inyección](lessons/10-robo-cookies-xss-avanzado.md) | 24/11/2026 | Hecha |
| 11 | [RBAC y primer contacto con OWASP ZAP](lessons/11-rbac-owasp-zap.md) | 26/11/2026 | Hecha |

## Writeups
- [Escaneo de Metasploitable](writeups/escaneo-metasploitable.md)
- [Explotación de vsftpd 2.3.4 con Metasploit](writeups/explotacion-vsftpd-metasploitable.md)
- [Cracking de contraseñas en Metasploitable](writeups/cracking-metasploitable.md)
- [Primeras vulnerabilidades web en Juice Shop](writeups/juice-shop-primeras-vulnerabilidades.md)
- [XSS y manipulación de JWT en Juice Shop](writeups/juice-shop-xss-jwt.md)
- [Robo de cookies vía XSS y comparación de puntos de inyección](writeups/juice-shop-robo-cookies-xss.md)
- [Control de acceso roto (RBAC) en la API](writeups/juice-shop-rbac-api.md)

## Cheatsheets
- [Guía rápida: "quiero hacer X → uso Y"](cheatsheets/00-guia-rapida.md)
- [nmap](cheatsheets/nmap.md)
- [Enumeración y búsqueda de vulnerabilidades](cheatsheets/enumeracion.md)

## Aviso legal
Todo el contenido se ha practicado en entornos propios o de entrenamiento
autorizados. Nunca se deben escanear ni atacar sistemas sin permiso por escrito.
