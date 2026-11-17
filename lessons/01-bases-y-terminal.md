# Lección 01: Bases, mentalidad y terminal

## Qué es el hacking ético
Hacer lo mismo que haría un atacante, pero **con permiso por escrito** y con el
objetivo de que la organización corrija sus fallos antes de que los explote alguien
malicioso. Lo que distingue a un profesional no es la técnica, sino el permiso y el
informe final.

## La tríada CIA
| Propiedad | Significa | Ataque que la rompe |
|---|---|---|
| Confidencialidad | Solo accede quien debe | Robo de una base de datos de clientes |
| Integridad | Los datos no cambian sin permiso | Alterar el importe de una transferencia |
| Disponibilidad | El servicio funciona cuando se necesita | DDoS, ransomware |

Cada vulnerabilidad afecta a una o varias de las tres, y en un informe siempre se indica cuál.

## Fases de un test de intrusión
1. **Reconocimiento**: recopilar información del objetivo.
2. **Enumeración**: detectar servicios y puertos abiertos.
3. **Explotación**: aprovechar una vulnerabilidad para entrar.
4. **Post-explotación**: ver hasta dónde se puede llegar.
5. **Informe**: documentar lo encontrado y cómo corregirlo.

## Regla de oro
Solo se ataca lo que es **mío** (laboratorio), de plataformas de entrenamiento o lo que
tengo **autorizado por escrito**. Sin permiso es delito, aunque no se cause daño.

## Comandos básicos de terminal
| Comando | Qué hace |
|---|---|
| `whoami` | Muestra el usuario actual |
| `pwd` | Muestra la carpeta actual |
| `ls` / `ls -la` | Lista archivos / con detalle y ocultos |
| `cd carpeta` | Entra en una carpeta (`cd ~` vuelve a la personal) |
| `mkdir nombre` | Crea una carpeta |
| `echo "texto" > archivo` | Escribe texto en un archivo |
| `cat archivo` | Muestra el contenido de un archivo |
| `man comando` | Abre el manual de un comando (salir con `q`) |

## Teclado en Kali
Kali arranca con teclado inglés (US). Para pasarlo a español: `setxkbmap es`
(temporal) o `sudo dpkg-reconfigure keyboard-configuration` (permanente).

## Qué he aprendido
Entendí que el hacking ético se diferencia de un ataque en el permiso y en el informe,
no en la técnica. Aprendí la tríada CIA y que cada vulnerabilidad se clasifica según
qué propiedad rompe: por ejemplo, un ransomware ataca sobre todo la disponibilidad.
También me quedó clara la estructura de un test de intrusión (reconocimiento,
enumeración, explotación, post-explotación e informe), que será el hilo de todo lo
demás. Y practiqué los comandos básicos de terminal, que es donde se trabaja el 90 %
del tiempo en Linux. Un detalle práctico: Kali arranca con teclado inglés y hay que
cambiarlo a español.
