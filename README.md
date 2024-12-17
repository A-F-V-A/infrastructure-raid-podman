# Configuración de Servidor con RAID y Volúmenes Lógicos

Este documento guía la configuración de un servidor en una máquina virtual con tres arreglos RAID 1, donde cada RAID estará compuesto por discos de 2GB, y se crearán volúmenes lógicos de 1GB a partir de estos.


## Índice
1. [Requisitos Previos](#requisitos-previos)
2. [Creación de la Máquina Virtual](#creación-de-la-máquina-virtual)
3. [Configuración de los Discos](#configuración-de-los-discos)
4. [Creación de RAIDs](#creación-de-los-raids)
5. [Creación de Volúmenes Físicos](#creación-de-los-volumenes-físicos)
6. [Creación de los Grupos de Volúmenes](#creación-del-grupo-de-volumenes)
7. [Configuración de Volúmenes Lógicos](#creación-de-los-volumenes-lógicos)
8. [Asignación de Formato](#darle-formato-a-los-volúmenes-lógicos)
9. [Montaje de Volúmenes](#montaje-de-volumenes)
10. [Instalación de Docker](#instalación-de-docker)
11. [Instalación de Podman](#instalación-de-podman)
12. [Creación de Dockerfile](#creación-de-dockerfile-para-cada-contenedor)
13. [Dockerfile Apache](#creación-de-dockerfile-apache-docker)
14. [Dockerfile MySQL](#creación-dockerfile-mysql-docker)
15. [Dockerfile Nginx](#creación-dockerfile-nginx-podman)
16. [Conclusión](#conclusión)

## Archivos Dockerfile
1. [Archivo Dockerfile Apache](./apache/Dockerfile)
2. [Archivo Dockerfile MySQL](./mysql/Dockerfile)
3. [Archivo Dockerfile Nginx](./nginx/Dockerfile)

## Archivos index.html
1. [index.html Apache](./apache/index.html)
2. [index.html Nginx](./nginx/index.html)
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

---

## Configuración de los Discos

Una vez que hayas agregado los discos a la máquina virtual, inicia la VM y configura los discos en el sistema operativo.

- Verifica que los discos estén detectados usando:
  ```bash
  ls /dev/
  ```
   Acá vamos a obtener la información de los discos, en este caso lo vemos de la siguiente manera: 

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-13-20.png" alt="discos" width="500" height="500">
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
      <img src="./assets/Screenshot from 2024-11-13 22-19-14.png" alt="raid" width="500" height="500">
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
      <img src="./assets/Screenshot from 2024-11-13 22-21-47.png" alt="fisico" width="500">
   </p>

   Ejecutamos los 3 comandos de creacion de los volumenes físicos y podemos verificar que al primer momento de crear nuestros tres volumenes, nos creo con éxito el primero, pero los otros dos nos genero un error, el cual fue que no usamos **sudo** para darle el permiso que necesitaba, después hicimos la creación de los 2 volumenes que nos faltaban y quedaron creados con éxito.

## Creación del Grupo de Volumenes

Es importante crear nuestro grupo de volúmenes, el cual lo podemos crear con el siguiente comando:
- Creación del Grupo de Volúmenes
   ```bash
   sudo vgcreate nombre-grupo /dev/md0
   ```

   Para nuestro proyecto es necesario crear 3 grupos de volúmenes, 1 para cada servicio: 
   <p aligin = 'center'>
       <img src="./assets/Screenshot from 2024-11-13 22-24-36.png" alt="grupo" width="500">
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
      <img src="./assets/Screenshot from 2024-11-13 22-30-17.png" alt="logico" width="500">
   </p>

   Acá podemos evidenciar que creamos 3 volúmenes lógicos, cada uno con 1G de espacio y los cuales los llamamos con la inicial **lvnombre** y estos fueron creacios éxitosamente

## Darle Formato a los Volúmenes Lógicos.

Es importante nosotros darle un formato, en este caso utilizaremos el formato mkfs.ext4, podremos usarlo con el siguiente comando:

- Formato para los Volúmenes Lógicos
   ```bash
   sudo mkfs.ext4 /dev/nombre-volumen-logico
   ```
   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-39-41.png" alt="logico" width="500">
   </p>

   A cada uno de los volúmenes lógicos que tenemos le damos el formato **mkfs.ext** para poder crear un sistema de archivos en ellos y así poder almacenar y organizar datos en el sistema de manera eficiente.

## Montaje de Volumenes

Es importante montar nuestro volumenes, para ello vamos a crear las carpetas correspondientes, para crear las carpetas lo podemos hacer con el siguiente comando:

- Creación de Carpetas
   ```bash
   sudo mkdir /mnt/nombre-carpeta
   ```
   
   En nuestro caso vamos a crear una carpeta por cada servicio que tengamos, en este caso 3 carpetas.

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-50-02.png" alt="logico" width="500">
   </p>

 Después de realizar la creación de las carpetas, montamos nuestros volumenes lógicos, para esto lo podemos hacer a partir del siguiente comando:

- Montaje de Vólumenes
   ```bash
   sudo mount /dev/pv-nombre/lv-nombre /mnt/nombre-carpeta-creada
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-53-11.png" alt="logico" width="500">
   </p>

   Aquí montamos cada volumen lógico en las carpetas creadas por cada servicio.

Con esto terminaríamos el montaje de los volúmenes, asegurando una estructura de almacenamiento eficiente y lista para el uso en el sistema, optimizando la organización y facilitando la administración de los datos.

---

## Instalación de Docker

Para poner iniciar con la instalación de Docker es importante primero irnos a la documentación oficial de Docker, aquí nos explicara que debemos hacer para la correcta instalación. Podemos verla en el siguiente enlace:  [Documentación Docker](https://docs.docker.com/engine/install/ubuntu/)

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-42-36.png" alt="logico" width="500">
   </p>

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-42-06.png" alt="logico" width="500">
   </p>

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-42-31.png" alt="logico" width="500">
   </p>

Es importante agregar **Docker** al grupo del usuario podemos realizar el siguiente comando:

- Agregar Docker al Grupo de Usuario
   ```bash
   sudo usermod -aG docker user
   ```
   Esto nos permitirá ejecutar comandos de Docker sin tener que usar **sudo** cada vez, facilitando el uso de Docker y haciendo el trabajo más eficiente.

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-43-40.png" alt="logico" width="600">
   </p>

## Instalación de Podman

Para la instalación de Podman, es necesario realizar un proceso similar al de Docker. Es fundamental que nos apoyemos en la documentación oficial de cada herramienta para asegurar una instalación correcta. En este caso, podemos consultar la documentación de Podman en el siguiente enlace: [Documentación Podman](https://podman.io/docs/installation)

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-44-21.png" alt="logico" width="500">
</p>

Después de instalar Docker y Podman, podemos ejecutar los siguientes comandos para verificar que ambas herramientas se han instalado correctamente:

- Versión Docker y Podman
   ```bash
   podman --version && docker --version
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 22-45-30.png" alt="logico" width="500">
   </p>

---

## Creación de DockerFile para cada Contenedor

Para comenzar con la creación del Dockerfile de cada servicio, primero es necesario crear una carpeta específica para cada uno. En estas carpetas almacenaremos el Dockerfile correspondiente y cualquier otro archivo necesario para configurar cada servicio.

- Comando para crear cada carpeta
   ```bash
   sudo mkdir nombre-carpeta
   ```

- Comando para crear un archivo

   ```bash
   sudo touch Dockerfile
   ```


## Creación de Dockerfile Apache (Docker)

Comenzamos creando la carpeta correspondiente para el servicio de Apache, junto con su archivo **Dockerfile**.

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-01-38.png" alt="logico" width="500">
</p>

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-01-45.png" alt="logico" width="500">
</p>

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-02-13.png" alt="logico" width="500">
</p>

el siguiente paso es modificar el **Dockerfile**, para esto agregaremos lo siguiente dentro del archivo.

- Dockerfile apache: 

   ```bash
   FROM ubuntu:latest

   RUN apt-get update && apt-get install -y apache2 && \
       apt-get clean && \
       rm -rf /var/lib/apt/lists/*
   
   EXPOSE 80

   CMD ["apache2ctl", "-D", "FOREGROUND"]
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-03-00.png" alt="logico" width="500">
</p>

   Este **Dockerfile** usa una imagen de Ubuntu y luego instala Apache, el servidor web. Después, limpia archivos innecesarios para reducir el tamaño de la imagen. Expone el puerto 80, que es el usado por los sitios web, y finalmente, al iniciar el contenedor, arranca Apache para que el servidor web esté funcionando y listo para recibir solicitudes. [Revisar Dockerfile](./apache/Dockerfile)

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-03-19.png" alt="logico" width="500">
</p>

Lo siguiente que vamos a hacer es construir la imagen a partir del Dockerfile que acabamos de crear.

- Comando para Construir la Imagen

   ```bash
  docker build -t my-apache apache/
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-10-03.png" alt="logico" width="500">
</p>

El siguiente paso consiste en ejecutar los contenedores y montarlos sobre los **volúmenes lógicos (LVM)**, asegurándonos de que estén correctamente configurados para utilizar el almacenamiento gestionado de manera eficiente. Para eso realizaremos lo siguiente:

- Comando para Ejecutar el Contenedor

   ```bash
   docker run -d --name my-apache -p 5051:80 -v /mnt/apache:/var/www/html my-apache
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-13-22.png" alt="logico">
   </p>

   Este comando podemos ejecutarlo para iniciar un contenedor en segundo plano con la imagen **my-apache**. Le asignamos el nombre **my-apache**, mapeamos el puerto **5051** del host al puerto **80** del contenedor **(-p 5051:80)**, y montamos el directorio **/mnt/apache** del host en **/var/www/html** del contenedor, lo que nos permite compartir archivos entre ambos. De esta manera, Apache se ejecutará en el contenedor y servirá el contenido desde el directorio especificado en el host.

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-14-13.png" alt="logico" width="500">
</p>

Como podemos ver, si accedemos a nuestra IP local a través del puerto **5051**, podremos ver el índice predeterminado de Apache que se está sirviendo desde el contenedor. Esto confirma que el contenedor está funcionando correctamente y que la configuración del mapeo de puertos y el montaje del volumen se ha realizado con éxito.

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-15-18.png" alt="logico" width="500">
</p>

Ahora, vamos a crear nuestro propio archivo index.html para mostrar un mensaje personalizado de "Hola" desde Apache. Para hacerlo, simplemente debemos crear un archivo HTML con el contenido deseado y colocarlo en el directorio que hemos montado previamente en el contenedor **/mnt/apache**

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-15-49.png" alt="logico" width="500">
</p>

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-16-37.png" alt="logico" width="500">
</p>

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-16-58.png" alt="logico" width="500">
</p>

Al modificar el archivo **/mnt/apache/index.html** en mi máquina local, los cambios se reflejan automáticamente en el navegador. Esto sucede porque monté el directorio local en el contenedor, por lo que cualquier ajuste en el archivo se muestra de inmediato al acceder a mi ip local con su pueto **5051**. [Revisar index.html](./apache/index.html)

## Creación Dockerfile MySQL (Docker)

Para la creación del **Dockerfile** de MySQL, el proceso es muy similar al que hicimos con Apache. Primero, debemos crear una carpeta donde alojaremos nuestro **Dockerfile**. Luego, dentro de esa carpeta, crearemos el archivo **Dockerfile** con las instrucciones ne
cesarias para configurar MySQL. 

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-19-29.png" alt="logico" width="500">
</p>

Con el siguiente comando podemos construir la imagen de MySQL:

   ```bash
   FROM mysql:latest

   ENV MYSQL_ROOT_PASSWORD=afvajdlm
   ENV MYSQL_DATABASE=myafvajdlm
   ENV MYSQL_USER=myuserafvajdml
   ENV MYSQL_PASSWORD=afvajdlm

   EXPOSE 3306
   ```
   Este **Dockerfile** lo creeamos utilizando la imagen oficial de **MySQL** más reciente como base. Establecimos la contraseña para el usuario root, creamos una base de datos llamada myafvajdlm y también un usuario adicional llamado myuserafvajdml con su respectiva contraseña. Además,se expuso el puerto **3306**, que es el predeterminado de **MySQL**, para permitir el acceso a la base de datos desde fuera del contenedor. Así, tengo una instancia de **MySQL** lista para usar con configuraciones personalizadas. [Revisar Dockerfile](./mysql/Dockerfile)

   Realizamos la construcción de la imagen a partir del Dockerfile que acabamos de crear.

- Comando para Construir la Imagen

   ```bash
  docker build -t my-mysql mysql/
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-22-21.png" alt="logico" width="500">
</p>

El siguiente paso consiste en ejecutar los contenedores y montarlos sobre los **volúmenes lógicos (LVM)**, asegurándonos de que estén correctamente configurados para utilizar el almacenamiento gestionado de manera eficiente. Para eso realizaremos lo siguiente:

- Comando para Ejecutar el Contenedor

   ```bash
   docker run -d --name my-mysql -p 3306:3306 -v /mnt/mysql:/var/lib/mysql my-mysql
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-25-53.png" alt="logico">
   </p>

Este comando podemos ejecutarlo para iniciar un contenedor en segundo plano con la imagen my-**mysql**. Le asignamos el nombre **my-mysql**, mapeamos el puerto **3306** del host al puerto **3306** del contenedor **(-p 3306:3306)**, y montamos el directorio **/mnt/mysql** del host en **/var/lib/mysql** del contenedor, lo que nos permite persistir los datos de la base de datos en nuestro sistema local para que no se pierdan al reiniciar el contenedor.

Ahora vamos a ejecurar MySQL, para poder hacerlo tenemos que realizar el siguiente comando.

- Comando para Ejecutar MySQL:
   ```bash
   docker run exec -it my-mysql -u myuserafvajdlm -p my-mysql
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-27-19.png" alt="logico">
   </p>

Después de ejecutar este comando, podemos evidenciar que **MySQL** nos pedirá la contraseña de inmediato. Esta contraseña es la misma que asignamos previamente en nuestro archivo **Dockerfile** durante la configuración de la imagen. Una vez ingresada correctamente, tendremos acceso a la consola de MySQL para interactuar con la base de datos.

Ahora podemos interactuar con la consola de MySQL, para poder verificarlo podemos hacer el siguiente comando

- Comando para 
   ```bash
   SHOW DATABASES
   ```
   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-29-29.png" alt="logico">
   </p>

Podemos evidenciar que el usuario que configuramos en el archivo **Dockerfile** está disponible. Ahora, solo sería cuestión de acceder a este usuario y comenzar a realizar pruebas. En este caso, accedimos al usuario y creamos una tabla llamada test_table con dos columnas: id, de tipo **INT** como clave primaria, y name, de tipo **VARCHAR(50)**. Esto confirma que el contenedor y la base de datos están funcionando correctamente.

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-31-33.png" alt="logico">
   </p>

Insertamos un dato en nuestra tabla test_table y luego realizamos una consulta sencilla para obtener y verificar los datos almacenados. Esto nos permitió confirmar que la base de datos está operativa y que la tabla funciona correctamente para almacenar y recuperar información.

Ahora solo para salir podemos poner **exit** para salir de la consola de MySQL

- Comando para ver los contenedores activos
   ```bash
   docker ps
   ```
   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-32-37.png" alt="logico">
   </p>

Con este comando podemos verificar que actualmente tenemos dos contenedores activos: **my-apache y my-mysql**. Esto demuestra que ambos servicios están en ejecución y listos para ser utilizados.

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-35-42.png" alt="logico">
</p>

Eliminamos por completo el contenedor y revisamos la carpeta **/mnt/mysql**, donde pudimos comprobar que la información de las acciones realizadas anteriormente se mantiene intacta. Esto es posible gracias al montaje de volúmenes, que asegura que los datos se guarden de manera persistente en el sistema local, incluso después de eliminar el contenedor.

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-37-05.png" alt="logico">
</p>

Para verificar que las acciones realizadas anteriormente siguen intactas, simplemente volvemos a ejecutar el contenedor, accedemos a la consola de **MySQL** y realizamos las mismas consultas que hicimos al inicio. Esto nos permite confirmar que los datos se han mantenido gracias al uso del volumen persistente.

## Creación Dockerfile Nginx (Podman)

Para la correcta implementación de **Nginx**, creamos una carpeta dedicada y dentro de ella un archivo **Dockerfile**. En este archivo definiremos las instrucciones necesarias para configurar nuestro servidor **Nginx** personalizado. Esto nos permitirá gestionar de manera eficiente las solicitudes HTTP y servir contenido estático o dinámico según nuestras necesidades.

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-45-33.png" alt="logico">
</p>

Al inicio, presentamos un problema al declarar el **Dockerfile** con ***FROM nginx:latest***. Esto se debe a que, al trabajar con Podman, no se puede acceder directamente al **Docker Hub**. Para solucionar este inconveniente, modificamos la declaración para que utilice un registro accesible por Podman, como un registro local o uno configurado específicamente para nuestra red. En este caso se declaro de la siguiente manera:

- Dockerfile Nginx

   ```bash
   FROM docker.io/library/nginx:latest

   EXPOSE 80

   CMD ["nginx", "-g", "daemon off;"]
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-45-33.png" alt="logico">
</p>

Aquí usamos directamente el enlace para que, mediante Podman, podamos acceder a la última versión de **Nginx** disponible en **Docker Hub**, evitando el problema de acceso directo. De esta forma, Podman puede obtener la imagen correctamente. En la imagen adjunta podemos evidenciar el error que se generó al intentar acceder sin especificar el enlace completo, lo que nos impidió descargar la imagen de forma correcta inicialmente. [Revisar Dockerfile](./nginx/Dockerfile)

Realizamos la contstrucción del dockerfile mediante podman, para esto usamos el siguiente comando:

   ```bash
   podman build -t my-nginx nginix/
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-46-13.png" alt="logico">
   </p>

   Este comando construye la imagen de **Nginx** utilizando el **Dockerfile** que hemos creado en el directorio actual. Al usar Podman, se generará una imagen etiquetada como **my-nginx** que podrá ser utilizada para crear contenedores.

Ahora ya construida la imagen podremos iniciar nuestro contenedor, para ello vamos a ejecutar los siguientes comandos

- Comando para la creación del contenedor
   ```bash
   podman run -d --name my-nginx -p 0.0.0.:5052:80 nginx
   ```
   Este comando ejecuta el contenedor en segundo plano **(-d)** usando la imagen **my-nginx** que acabamos de construir. Le asignamos el nombre **my-nginx**, y mapeamos el puerto **8080** del host al puerto **80** del contenedor, lo que nos permite acceder a Nginx a través de la dirección localhost:8080.

- Comando para el tráfico de puertos
   ```bash
   sudo ufw allow 8080/tcp && sudo ufw allow 80/tcp
   ```

- Comando para la configuración del firewall
   ```bash
   sudo ufw reload
   ```
   
   Con estos comandos, configuramos el tráfico de puertos en nuestro sistema, permitiendo que las conexiones entrantes a los puertos 8080 (para Nginx) y 80 (puerto HTTP estándar) sean aceptadas. Esto asegura que podamos acceder a los servicios en esos puertos desde otras máquinas en la red, mientras mantenemos el control sobre el tráfico de otros puertos a través del firewall UFW.

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-14 00-03-53.png" alt="logico">
   </p>

- Comando para iniciar el contenedor

   ```bash
   podman run -d --name my-nginx -p 8080:80 -v /mnt/nginx:/usr/share/nginx/html my-nginx
   ```
   Este comando inicia el contenedor my-nginx utilizando Podman, asignando el puerto 8080 del host al puerto 80 del contenedor, lo que nos permite acceder a Nginx a través de localhost:8080. Además, mapeamos el directorio /mnt/nginx del host al directorio /usr/share/nginx/html del contenedor, donde Nginx busca los archivos HTML para servir. Esto nos permite tener un index.html personalizado y hacer cambios en el contenido directamente desde el host.

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-14 00-06-54.png" alt="logico">
   </p>


Después de tener todo configurado, podemos acceder desde nuestro navegador a **localhost:8080** (o la IP de tu máquina si estás en otra red). Al hacerlo, veremos el saludo inicial de Nginx, que es la página predeterminada que aparece cuando se instala y ejecuta Nginx por primera vez. Esto indica que el contenedor está funcionando correctamente y que el servidor web está activo y respondiendo en el puerto configurado.

 <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-14 00-05-52.png" alt="logico">
   </p>

Ya finalmente hacemos la creación de nuestro index.html para que tengamos un saludo personalizado, para eso lo creamos en la siguiente ruta

- Creación de nuestro index.html
   ```bash
   sudo touch /mnt/nginx/index.html
   ```

   <p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-13 23-50-28.png" alt="logico">
   </p>

   Con este comando hacemos la creación de nuestro **index.html**, el cual nos va a permitir personalizar la página de inicio de nuestro servidor Nginx. Este archivo contendrá el contenido que deseamos mostrar cuando accedemos al servidor desde el navegador, reemplazando el saludo predeterminado de Nginx con nuestro propio mensaje o diseño. [Ver index.html](./nginx/index.html)

<p aligin = 'center'>
      <img src="./assets/Screenshot from 2024-11-14 00-07-18.png" alt="logico">
</p>

Finalmente, podemos ver que se cargó satisfactoriamente nuestro index.html en el servidor Nginx y cualquier cambio realizado en el archivo se reflejará de inmediato en la página al hacer las solicitudes al servidor. Esto se debe a que estamos mapeando el directorio /mnt/nginx del host al contenedor, lo que facilita la modificación del contenido sin necesidad de reconstruir el contenedor cada vez que realizamos un cambio.

---

## Conclusión 

podemos concluir que hemos logrado configurar correctamente nuestros contenedores utilizando **Podman** y **Docker**, implementando los servicios de **Apache**, **MySQL** y **Nginx** con éxito. A través de la creación y personalización de los respectivos Dockerfiles, pudimos construir las imágenes necesarias, iniciar los contenedores, y realizar pruebas funcionales, como la carga de archivos personalizados en Nginx y la gestión de bases de datos en MySQL.
Además, configuramos adecuadamente las reglas del firewall para permitir el tráfico en los puertos necesarios, lo que nos permitió acceder a nuestros servicios desde el navegador sin inconvenientes.

Una parte fundamental de esta implementación fue el uso de **LVM**, que nos permitió gestionar de manera eficiente y flexible el almacenamiento de los contenedores y servicios. Con **LVM**, pudimos organizar y redimensionar los volúmenes lógicos de manera dinámica, lo que proporciona una mayor escalabilidad y control sobre los recursos de almacenamiento en el sistema. Esta práctica mejora la resiliencia y el manejo del almacenamiento en entornos de contenedores, permitiendo adaptarnos fácilmente a futuros requerimientos y optimizando el rendimiento de los servicios.

En conclusión, hemos logrado implementar con éxito un entorno de contenedores utilizando Podman y Docker, configurando y ejecutando los servicios de Apache, MySQL y Nginx. A lo largo del proceso, aprendimos a crear y personalizar Dockerfiles, gestionar contenedores y realizar pruebas funcionales. También configuramos el acceso remoto y optimizamos el tráfico de puertos mediante reglas de firewall.

El uso de LVM (Logical Volume Management) fue clave para gestionar el almacenamiento de manera eficiente, proporcionando flexibilidad y escalabilidad, lo cual es crucial en entornos de contenedores. Esta implementación no solo mejora el manejo del almacenamiento, sino que también facilita la administración dinámica de volúmenes y recursos en el sistema.

Con todo esto, hemos establecido una infraestructura flexible, escalable y fácil de mantener, con la capacidad de adaptarnos a cambios y nuevas demandas de manera eficiente.
















