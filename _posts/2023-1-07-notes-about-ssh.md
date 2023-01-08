---
layout: single
title: Notas de investigación sobre SSH
date: 2023-01-7
classes: wide
header:
  teaser: /assets/images/notes-about-ssh/teaser.jpg
categories:
  - HowTo
  - Linux
tags:
  - Networking
  - OpenSSH server
---


Me despertó la curiosidad saber más sobre cómo funciona SSH y decidí crear este post para profundizar mis conocimientos y tener una guía de referencia para el futuro. Aunque ya tenía una noción básica de su funcionamiento, quería entender mejor cómo se aplica en la práctica y cuáles son sus principales usos y beneficios. ¡Espero que disfruten leyendo sobre este interesante tema y aprendan tanto como yo!

# Instalación y configuración

Empezamos con los comandos clasicos para instalar OpenSSH server en nuestra máquina linux

<details>
  <summary>Explicación del comando</summary>
  <p>
    Actualiza los repositorios del sistema y revisa si hay alguna actualizacion disponible
  </p>
</details>

``` bash
foo@bar:~$ sudo apt-get update
```
<details>
  <summary>Explicación del comando</summary>
  <p>
    Descarga e instala las actualizaciones para cada paquete
  </p>
</details>

``` bash
foo@bar:~$ sudo apt-get upgrade
```

A la hora de querer instalar openssh-server nos puede surgir la duda de porque hay tantos paquetes openssh.
<details>
  <summary>Explicación</summary>
  <p>
  <li>openssh-client: Este paquete solo sirve iniciar una conexión</li>
  <li>openssh-server: Este paquete sirve para aceptar las conexiones por el puerto 22.</li>
  <li>openssh-client-ssh1: Similar al openssh-client pero a diferencia del primero este utiliza el protocolo ssh1 el cual es inseguro debido a que cuatro de los algoritmos de encriptacion que utiliza son inseguros pero se siguen utilizando porque la mayoria de sistemas linux busca ser lo más compatible posible, con tecnologías antiguas además, al siempre no tener acceso a internet</li>
  <li>openssh-know-hosts: Este paquete le permite descargar claves de host públicas de múltiples fuentes, filtrar los nombres de host que vienen con ellas y fusionarlas en un solo archivo para su uso por OpenSSH</li>
  <li>openssh-sftp-server: Este paquete proporciona el módulo del servidor SFTP para el servidor SSH. Es necesario si quieres acceder a tu servidor SSH con SFTP. El módulo del servidor SFTP también funciona con otros daemon SSH como Dropbear.</li>
  <li>openssh-tests: Este paquete proporciona el conjunto de pruebas de regresión de OpenSSH. Está pensado principalmente para su uso con el sistema AutoPkgTest</li>   <br>

  Como dato extra tambien podemos encontrar el paquete ssh el cual instala OpenSSH-client y OpenSSH-server
  </p>
</details>

![](/assets/images/notes-about-ssh/1.png)


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
    ufw tambien llamado Uncomplicated Firewall como su nombre lo dice fue diseñado para ser de facil uso, su utilidad está en simplificar la tarea de configurar las iptables.
  </p>
</details>

# Medidas de seguridad 
1. Lo primera obviamente es utilizar una contraseña segura puesto que al día ocurren miles de ataques de fuerza bruta destinados a los servidores SSH.

2. Cuando abrimos una conexion SSH a veces nos olvidamos de cerrarla, por lo que es bueno establecer un tiempo limite de inactividad. Esto lo podemos configurar modificando los siguientes parámetros del archivo /etc/ssh/sshd_config.

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

3. Desactivar el login sin contraseña editando parámetro del archivo /etc/ssh/sshd_config.

``` bash
PermitEmptyPasswords no
```

4. Añadir una lista blanca de usuarios permitidos para conectarse mediante ssh modificando el siguiente parámetro del archivo /etc/ssh/sshd_config.


``` bash
AllowUsers user1 user2
```

Para que los cambios hagan efecto debes reiniciar el servicio de la siguiente manera:

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

7. Para la autenticación, utilizamos llaves publicas y privadas en vez de contraseñas.
Desde nuestra máquina cliente creamos una llave pública y una llave privada de la siguiente manera:


``` bash
ssh-keygen -t rsa
``` 
Nos creará 2 archivos: id_rsa y id_rsa.pub. </br>

El archivo id_rsa.pub debes enviarlo al servidor, especificamente debemos pegar su contenido en el archivo ~/.ssh/authorized_keys del servidor.
Debes darle permiso al archivo

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

Esta es la parte que más me ha interesado investigar durante este post ya que ahondamos en los fundamentos de SSH desde los tipos de encriptación hasta la manera en que realiza la conexión

Para explicarlo bien y que sea entendible, vamos a explicarlo de manera general y luego ir profundizando más.

El protocolo de SSH consiste en 3 partes principales:

1. Verificación del servidor: En esta parte, nosotros (el cliente) vamos a verificar si el servidor al cual nos queremos conectar mediante SSH es auténtico. Esto se hace porque es posible suplantar un servidor SSH, es decir, cuando nos conectamos a un servidor A, un atacante puede hacerse pasar por el servidor A. Esto se puede evitar gracias al archivo known_hosts que nos indica si el servidor al que nos queremos conectar es conocido o no.

2. Generación de una clave de sesión para cifrar toda la comunicación: Durante esta fase, el servidor y el cliente se van a poner de acuerdo para que mediante unos algoritmos que pactaron antes van a generar una clave de sesión compartida y un ID de sesión.

3. Autenticación del cliente: Hasta este punto, la conexión ya es segura pues el cliente y el servidor ahora comparten una clave temporal simétrica para cifrar y descifrar (clave de sesión) y el último paso sería autenticarse en el servidor que puede realizarse mediante contraseña, llave pública o ambas.

Ya que dimos un vistazo global del protocolo SSH, ahora profundizaremos un poco más guiándonos en el siguiente gráfico.

![](/assets/images/notes-about-ssh/2.png)

1. Establece la conexión: Se refiere al ya conocido TCP 3-way Handshake. Si no lo conoces, en un futuro realizaré un post teórico acerca de él y otros protocolos más.   
 ![](/assets/images/notes-about-ssh/3.png)

2. Servidor envía llave pública al cliente: Una vez que establecemos la conexión, el servidor nos enviará su llave pública.

3. Verifica la identidad del servidor: El cliente procederá a revisar en sus archivos (known_hosts).  
![](/assets/images/notes-about-ssh/4.png)
Si el servidor es conocido (es decir si ya nos hemos
conectando antes al servidor).   
![](/assets/images/notes-about-ssh/5.png)

4. Negociación de versiones: El cliente y el servidor deciden qué versión de SSH utilizar, ya sea 1.0 o 2.0. La diferencia es que dependiendo de la versión habrá más métodos de autenticación, métodos de intercambio de claves y mejoras en las capacidades del servicio.   
![](/assets/images/notes-about-ssh/6.png)

5. Negocio de algoritmos: En esta parte se negocian algunos algoritmos como: el algoritmo de intercambio de claves para generar claves de sesión, un algoritmo de cifrado para cifrar datos, un algoritmo de clave pública para firma digital y autenticación, y un algoritmo HMAC para proteger la integridad de los datos.   
![](/assets/images/notes-about-ssh/7.png)   

6. Intercambio de llaves (Key Exchange): Cada uno por separado utiliza los datos que compartieron y sus datos privados para que cada uno, por su cuenta, mediante el algoritmo de Diffie-Hellman compute una clave de sesión idéntica, la cual se utilizará para cifrar y descifrar los datos.   
![](/assets/images/notes-about-ssh/8.png)   

7. Autenticación del cliente: En este punto la conexión ya es segura y cifrada, ahora solo quedaría autenticarse mediante una llave pública o una contraseña.
![](/assets/images/notes-about-ssh/9.png)   
Espero que este post te haya ayudado a comprender de mejor manera el protocolo SSH. Si hay algo más que agregar, lo haré en un futuro. ¡Hasta pronto!