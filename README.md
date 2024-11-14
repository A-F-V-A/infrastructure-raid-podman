# Configuración de Servidor con RAID y Volúmenes Lógicos

Este documento te guiará en la configuración de un servidor en una máquina virtual con tres arreglos RAID 1, donde cada RAID estará conformado por discos de 2GB, y se crearán volúmenes lógicos de 1GB a partir de estos.

## Índice
1. [Requisitos Previos](#requisitos-previos)
2. [Creación de la Máquina Virtual](#creación-de-la-máquina-virtual)
3. [Configuración de los Discos](#configuración-de-los-discos)
4. [Creación de RAIDs](#creación-de-raids)
5. [Configuración de Grupos de Volúmenes y Volúmenes Lógicos](#configuración-de-grupos-de-volúmenes-y-volúmenes-lógicos)

---

## Requisitos Previos

1. **Máquina Virtual**: Configurada en tu software de virtualización (ej. VirtualBox, VMware).
2. **Sistema Operativo**: Distribución de Linux compatible con administración de RAID (ej. Ubuntu, CentOS).
3. **Herramientas necesarias**:
   - `mdadm`: Para la configuración de RAID.
   - `lvm2`: Para la administración de volúmenes lógicos.

## Creación de la Máquina Virtual

1. Inicia tu software de virtualización y crea una nueva máquina virtual.
2. Asigna al menos 4GB de RAM y 10GB de almacenamiento para el sistema base.
3. Agrega **seis discos adicionales de 2GB** cada uno para usarlos en la configuración RAID.

## Configuración de los Discos

Una vez que hayas agregado los discos a la máquina virtual, inicia la VM y configura los discos en el sistema operativo.

- Verifica que los discos estén detectados usando:
  ```bash
  lsblk
  ```