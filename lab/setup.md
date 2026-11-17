# Configuración del laboratorio

Laboratorio aislado para practicar hacking ético de forma legal y segura.

## Componentes
| Máquina | Rol | Detalles |
|---|---|---|
| Kali Linux (VM prefabricada para VirtualBox) | Atacante | 4 GB RAM, 2-4 CPUs |
| Metasploitable 2 | Objetivo vulnerable | 512 MB RAM, disco `.vmdk` existente |

## Red
Red NAT de VirtualBox llamada `LabHacking` (rango `10.0.2.0/24`, DHCP activado).
- Las VMs se ven entre sí y tienen internet.
- Mi red doméstica **no** ve las máquinas vulnerables.
- Nunca se usa modo puente (Bridged) con máquinas vulnerables.

## Pasos
1. Instalar VirtualBox.
2. Importar Kali (`.vbox` descargado de kali.org) y actualizar:
   `sudo apt update && sudo apt full-upgrade -y`
3. Configurar teclado: `setxkbmap es`.
4. Crear la Red NAT: Archivo → Herramientas → Gestor de red → Redes NAT.
5. Crear la VM Metasploitable2 (Linux, Ubuntu 32-bit, 512 MB) usando el `.vmdk` existente
   (en el selector, filtro "Todos los archivos").
6. Activar I/O APIC (Sistema → Placa base).
7. Adaptador 1 de ambas VMs: Red NAT `LabHacking`.
8. Verificar con `ping` entre ambas y hacer instantáneas (snapshots).

## Notas
- Credenciales de Metasploitable: públicas (`msfadmin`), sin riesgo.
- Los discos de VM no se suben al repositorio (ver `.gitignore`).
