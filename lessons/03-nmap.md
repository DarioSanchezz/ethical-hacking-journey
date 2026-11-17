# Lección 03: Reconocimiento con nmap

> Solo se escanea mi laboratorio (Red NAT `LabHacking`). Nunca redes ajenas sin autorización.

## Qué es y para qué sirve
**nmap** (*Network Mapper*) descubre equipos, puertos abiertos, servicios, versiones y
sistema operativo. Es el punto de partida de cualquier test de intrusión y también una
herramienta defensiva para saber qué está expuesto.

## Estados de un puerto
| Estado | Significado |
|---|---|
| open | Hay un servicio escuchando (respondió SYN-ACK) |
| closed | Accesible pero sin servicio (RST) |
| filtered | Un firewall bloquea el paso |
| open\|filtered | No puede distinguirlo (típico en UDP) |

## Comandos y opciones
| Comando | Qué hace |
|---|---|
| `nmap -sn RED/24` | Solo descubre equipos vivos (sin escanear puertos) |
| `nmap IP` | Escaneo básico de los 1000 puertos más comunes |
| `sudo nmap -sS IP` | Escaneo SYN (medio abierto), el habitual |
| `-sT` | Escaneo TCP connect (sin privilegios, más ruido) |
| `-p 21,22,80` / `-p 1-1000` / `-p-` | Puertos concretos / rango / todos (1-65535) |
| `-F` / `--top-ports N` | 100 más comunes / los N más comunes |
| `-sV` | Detecta servicio y versión |
| `-O` | Detecta el sistema operativo |
| `-sC` | Scripts NSE por defecto |
| `--script nombre` | Ejecuta un script concreto (`ftp-anon`, `vuln`) |
| `-A` | Equivale a `-sV -sC -O --traceroute` (muy ruidoso) |
| `-T0` a `-T5` | Velocidad (T4 en laboratorio) |
| `-v` | Más información mientras escanea |
| `-oN` / `-oX` / `-oA` | Guarda en texto / XML / los tres formatos |
| `-sU` | Escaneo UDP (lento, combinar con `--top-ports`) |

## Por qué `-p-` importa
Por defecto nmap no mira los 65535 puertos; un servicio en un puerto raro pasaría
desapercibido.

## Flujo de trabajo
1. `nmap -sn 10.0.2.0/24` → localizar Metasploitable.
2. `nmap IP` → vista rápida.
3. `sudo nmap -sS -p- -T4 IP` → todos los puertos.
4. `sudo nmap -sC -sV -O -oA escaneo_meta IP` → versiones, scripts y SO, guardado.

El escaneo que ejecuté contra mi Metasploitable (10.0.2.3) fue:
`sudo nmap -sC -sV -oA escaneo_meta 10.0.2.3` (resultado en [lab/scans/](../lab/scans/escaneo_meta.nmap)).

## Qué he aprendido
Aprendí que el reconocimiento es el primer paso real de un test de intrusión y que
nmap es la herramienta estándar. Entendí los estados de un puerto y cómo se relacionan
con el saludo TCP. Vi que el escaneo por defecto solo mira 1000 puertos y que hay que
usar `-p-` para no dejarse servicios en puertos raros. Lo más valioso fue `-sV`: la
versión de un servicio es lo que permite saber si es vulnerable. También aprendí a
combinar `-sC` y `-sV`, a elegir la velocidad con `-T`, y que conviene guardar siempre
los resultados con `-oA`, porque son la evidencia y la base del informe. Y entendí por
qué `-A` es cómodo en laboratorio pero demasiado ruidoso en una auditoría real.
