# Configuración de Servidor con RAID y Volúmenes Lógicos

Este documento guía la configuración de un servidor en una máquina virtual con tres arreglos RAID 1, donde cada RAID estará compuesto por discos de 2GB, y se crearán volúmenes lógicos de 1GB a partir de estos.

## Índice
1. [Requisitos Previos](#requisitos-previos)
2. [Creación de la Máquina Virtual](#creación-de-la-máquina-virtual)
3. [Configuración de los Discos](#configuración-de-los-discos)
4. [Creación de RAIDs](#creación-de-los-raids)
5. [Configuración de Grupos de Volúmenes y Volúmenes Lógicos](#configuración-de-grupos-de-volúmenes-y-volúmenes-lógicos)

---

## Requisitos Previos

1. **Máquina Virtual**: Configurada en tu software de virtualización (ej. VirtualBox, VMware).
2. **Sistema Operativo**: Distribución de Linux compatible con administración de RAID (ej. Ubuntu, CentOS).
3. **Herramientas necesarias**:
   - `mdadm`: Para la configuración de RAID.
   - `lvm`: Para la administración de volúmenes lógicos.

## Creación de la Máquina Virtual

1. Inicia tu software de virtualización y crea una nueva máquina virtual.
2. Asigna al menos 4GB de RAM y 10GB de almacenamiento para el sistema base.
3. Agrega **seis discos adicionales de 2GB** cada uno para usarlos en la configuración RAID.

## Configuración de los Discos

Una vez que hayas agregado los discos a la máquina virtual, inicia la VM y configura los discos en el sistema operativo.

- Verifica que los discos estén detectados usando:
  ```bash
  ls /dev/
  ```
   Acá vamos a obtener la información de los discos, en este caso lo vemos de la siguiente manera: 

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-13-20.png" alt="Discos" width="500" height="500">
   </p>

   Donde podemos observar que tenemos que tenemos los discos listos para la creación de los 3 Raid, en este caso tenemos **sdb, sdc, sdd, sde, sdf, sdg** los cuales son los discos funcionales que tenemos para la creación de nuestro Raid.

## Creación de los Raids

Una vez que hayamos verificado los discos, procedemos con la creación de cada uno de los raids, en este caso usaremos el siguiente comando.

- Creación de Raid Level 1
 ```bash
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/disco-asignado-1 /dev/disco-asignado-2
```
En este caso vamos a realizar la creación de cada uno de los raids:

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-19-14.png" alt="Discos" width="500" height="500">
   </p>

Ejecutados los 3 comandos vamos a tener creados nuestros Raids de nivel 1, cada uno va contener dos discos los cuales nos brindará un mejor respaldo de información.

## Creación de los Volumenes Físicos

Después de tener los Raids creados vamos a crear nuestros volumenes Físicos, los cuales debemos especificar donde estan nuestros Raids, ejemplo en el siguiente comando.

- Creación de los Volumenes Físicos
```bash
sudo pvcreate /dev/md0 && sudo pvcreate /dev/md1 && sudo pvcreate /dev/md0
```
Acá creamos en una sola linea de comando los 3 Volumenes Físicos:
  <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-21-47.png" alt="Discos" width="500">
   </p>

   Ejecutamos los 3 comandos de creacion de los volumenes físicos y podemos verificar que al primer momento de crear nuestros tres volumenes, nos creo con éxito el primero, pero los otros dos nos genero un error, el cual fue que no usamos **sudo** para darle el permiso que necesitaba, después hicimos la creación de los 2 volumenes que nos faltaban y quedaron creados con éxito.

## Creación del Grupo de Volumenes

Es importante crear nuestro grupo de volumenes, el cual lo podemos crear con el siguiente comando:

- Creación del Grupo de Volumenes
```bash
sudo vgcreate nombre-grupo /dev/md0
```

Para nuestro proyecto es necesario crear 3 grupo de volumenes, 1 para cada servicio: 
 <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-24-36.png" alt="Discos" width="500">
   </p>

En este caso creamos los 3 servicios para cada uno, en este caso para el apache, mysql y nginix y evidenciamos que quedaron creados exitosamente.

## Creación de los Volumenes Lógicos

Es importante crear los volumenes lógicos ya que nos ayudaran a gestionar de manera más flexible y eficiente el almacenamiento en un sistema. Para poder crealo debemos usar el siguiente comando:

- Creación de Volumenes Lógicos
```bash
sudo lvcreate -L 1G -n nombre-volumen-logico nombre-grupo-volumenes
```
Nosotros vamos a crear 3 volúmenes lógicos para cada servicio, entonces lo realizamos de la siguiente manera.

 <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-30-17.png" alt="Discos" width="500">
   </p>
Acá podemos evidenciar que creamos 3 volúmenes lógicos, cada uno con 1G de espacio y los cuales los llamamos con la inicial **lv-nombre del servicio**.
