---
layout: single
title: Crear un servidor ssh en casa 
date: 2022-11-19
classes: wide
header:
  teaser: /assets/images/slae32.png
categories:
  - HowTo
  - Linux
tags:
  - Networking
  - OpenSSH server
---

# Instalación y configuración

En el siguiente post veremos todo lo que investigue sobre como crear un servidor ssh en casa. Empezamos con los comandos clasicos para instalar OpenSSH server en nuestra máquina linux

<details>
  <summary>Explicacion del comando</summary>
  <p>
    Actualiza los repositorios del sistema y revisa si hay alguna actualizacion disponible
  </p>
</details>

``` bash
foo@bar:~$ sudo apt-get update
```
<details>
  <summary>Explicacion del comando</summary>
  <p>
    Descarga e instala las actualizaciones para cada paquete
  </p>
</details>

``` bash
foo@bar:~$ sudo apt-get upgrade
```

A la hora de querer instalar openssh-server nos puede surgir la duda de porque hay tantos paquetes openssh.
<details>
  <summary>Explicacion</summary>
  <p>
  <li>openssh-client: Este paquete solo sirve iniciar una conexión</li>
  <li>openssh-server: Este paquete sirve para aceptar las conexiones por el puerto 22.</li>
  <li>openssh-client-ssh1: Similar al openssh-client pero a diferencia del primero este utiliza el protocolo ssh1 el cual es inseguro debido a que cuatro de los algoritmos de encriptacion que utiliza son inseguros pero se siguen utilizando porque la mayoria de sistemas linux busca ser lo más compatible posible con tecnologías antiguas además, al no siempre tener acceso a internet</li>
  <li>openssh-know-hosts: Este paquete le permite descargar claves de host públicas de múltiples fuentes, filtrar los nombres de host que vienen con ellas y fusionarlas en un solo archivo para su uso por OpenSSH</li>
  <li>openssh-sftp-server: Este paquete proporciona el módulo de servidor SFTP para el servidor SSH. Es necesario si quieres acceder a tu servidor SSH con SFTP. El módulo del servidor SFTP también funciona con otros daemon SSH como dropbear.</li>
  <li>openssh-tests: Este paquete proporciona el conjunto de pruebas de regresión de OpenSSH. Está pensado principalmente para su uso con el sistema autopkgtest</li>   <br>

  Como dato extra tambien podemos encontrar el paquete ssh el cual instala openssh-client y openssh-server
  </p>
</details>

![](/assets/images/create-server-ssh/1.png)


Si no te surge la duda ejecute el siguiente comando para instalar el servidor openssh

``` bash
foo@bar:~$ sudo apt-get install openssh-server
```
Ahora revisamos si el servicio ssh se encuentra corriendo en segundo plano con el comando systemctl el cual controla el systemd y servicios.

<details>
  <summary>Dato</summary>
  <p>
     A este tipo de servicios corriendo en segundo plano se les llama Demonios o Daemons.
  </p>
</details>

``` bash
foo@bar:~$ sudo systemctl status ssh
```
En caso de no encontrarse corriendo deberás activarlo así

``` bash
foo@bar:~$ sudo systemctl enable --now ssh
```

Si tiene habilitado el firewall en ubuntu (ufw) entonces deberá permitir las conexiones entrantes para ssh

``` bash
foo@bar:~$ sudo ufw allow ssh
```

Si el servicio ssh se encuentra corriendo en otro puerto debe especificarlo de la siguiente manera en el firewall.

``` bash
foo@bar:~$ sudo ufw allow 22
```

<details>
  <summary>Dato</summary>
  <p>
    UFW tambien llamado Uncomplicated Firewall como su nombre lo dice fue diseñado para ser de facil uso, su utilidad está en simplificar la tarea de configurar las iptables.
  </p>
</details>

# Medidas de seguridad 
1. Lo primera obviamente es utilizar una contraseña segura puesto que al día ocurren miles de ataques de fuerza bruta destinados a los servidores SSH.

2. Cuando abrimos una conexion SSH a veces nos olvidamos de cerrarla, por lo que es bueno establecer un tiempo limite de inactividad. Esto lo podemos configurar modifcando los siguientes parametros del archivo /etc/ssh/sshd_config

``` bash
  ClientAliveInterval 360
  ClientAliveCountMax 0
```
Esto quiere decir que en 360 segundos de inactividad la conexion se cerrara.
<details>
  <summary>Dato</summary>
  <p>
  <li>ClientAliveInterval: Indica los segundos hasta que el servidor envie un paquete nulo al cliente para verificar si sigue activo.</li>
  <li>ClientAliveCountMax: Indica el numero de paquetes nulos que enviará el servidor, antes de finalizar la conexion</li>
  </p>
</details>

3. Desactivar el login sin contraseña editando parametro del archivo /etc/ssh/sshd_config

``` bash
PermitEmptyPasswords no
```

4. Añadir una lista blanca de usuarios permitidos para conectarse mediante ssh modificando el siguiente parametro del archivo /etc/ssh/sshd_config


``` bash
AllowUsers user1 user2
```

Para que los cambios hagan efecto deberemos reiniciar el servicio de la siguiente manera:

``` bash
service sshd restart
```

5. Desactivamos el logeo a SSH como root.<br/>

En  /etc/ssh/sshd_config escribimos:

``` bash
PermitRootLogin no
```
Y reiniciamos el servicio ssh.

``` bash
service sshd restart
```

6. Cambiamos el puerto en el cual se va ejecutar el servicio SSH. Esto lo hacemos porque mayormente los bots maliciosos atacan el puerto 22 de manera predeterminada.</br>

En /etc/ssh/sshd_config escribimos:

``` bash
Port 2025
```

Y reiniciamos el servicio ssh.

``` bash
service sshd restart
```

7. Para la autenticacion, utilizamos llaves publicas y privadas en vez de contraseñas.
Desde nuestra máquina cliente creamos una llave pública y una llave privada de la siguiente manera:


``` bash
ssh-keygen -t rsa
``` 
Nos creará 2 archivos: id_rsa y id_rsa.pub. </br>

El archivo id_rsa.pub deberemos mandarlo al servidor, especificamente debemos pegar su contenido en el archivo ~/.ssh/authorized_keys del servidor.
Le damos permiso al archivo

``` bash
chmod 600 ~/.ssh/authorized_keys
``` 
Y por ultimo en /etc/ssh/sshd_config escribimos:
``` bash
PasswordAuthentication no
```
8. Limite los intentos de inicio de sesión fallidos configurando el parámetro "MaxAuthTries" en el archivo de configuración de SSH (/etc/ssh/sshd_config).

9. Aplique parches y actualizaciones de seguridad de forma regular para asegurarse de tener la última protección contra vulnerabilidades conocidas.

# Como funciona SSH

Esta es la parte que más me ha intersado investigar durante este post ya que ahondamos en los fundamentos de SSH desde los tipos de encriptacion hasta la manera en como realiza la conexión.