---
layout: single
title: Notas de investigación sobre SSH de hiteek
date: 2023-1-7
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


El presente post lo cree porque me dio curiosidad saber el funcionamiento de este protocolo y queria entenderlo más a fondo, también me servirá como notas para recordar. Pero de igual manera le puse un a la escritura un tono un poco robotico al inicio pero luego narrativo.
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

Para explicarlo bien y que sea entendible vamos a explicarlo de manera general y luego ir profundizando más.

El proceso de SSH consiste en 3 partes principales:

1. Verificacion del servidor: En esta parte nosotros (el cliente) vamos a verificar si el servidor al cual nos queremos conectar mediante SSH es auténtico. Esto se hace porque es posible suplantar un servidor SSH, es decir cuando nos conectamos a un servidor A, un atacante puede hacerse pasar por el servidor A. Esto se puede evitar gracias al archivo known_hosts que nos indica si el servidor al que nos queremos conectar es conocido o no.

2. Generación de una clave de sesión para cifrar toda la comunicación: Durante esta fase el servidor y el cliente se van a poner de acuerdo para que mediante unos algoritmos que pactaron antes van a generar una clave de sesión compartida y un ID de sesión.

3. Autenticacion del cliente: Hasta este punto la conexion ya es segura pues el cliente y el servidor ahora comparten una clave temporal simetrica para cifrar y descifrar (clave de sesión) y el ultimo paso seria autenticarse en el servidor que puede realizarse mediante contraseña, llave publica o ambas.

Ya que dimos un vistaso global del protoclo SSH ahora profundizaremos un poco más guiandonos en el siguiente gráfico.

![](/assets/images/create-server-ssh/2.png)

1. Establece la conexión: Refiere al ya conocido TCP 3-way Handhsake por si no lo conoces en un futuro realizare un post teorico acerca de el mismo y otros protocolos más.

2. Servidor envía llave pública al cliente: Una vez que establecemos la conexión, el servidor nos enviará su llave pública.

3. Verifica la identidad del servidor: El cliente procederá a revisar en sus archivos (known_hosts) si el servidor es conocido (es decir si ya nos hemos conectando antes al servidor).

4. Negocio de versiones: El cliente y el servidor deciden que version de SSH utilizar si 1.0 o 2.0 y la diferencia es que dependiende de la version habrán más métodos de autenticación, métodos de intercambio de claves y mejora las capacidades del servicio.

5. Negocio de algoritmos: En esta parte se negocian algunos algoritmos como: algoritmo de intercambio de claves para generar claves de sesión, un algoritmo de cifrado para cifrar datos, un algoritmo de clave pública para firma digital y autenticación, y un algoritmo HMAC para la protección de la integridad de los datos.

6. Intercambio de llaves (Key Exchange): Cada uno por separado utiliza los datos que compartieron y sus datos privados para que cada uno por su cuenta mediante el algoritmo de Diffie-Hellman compute una clave de sesión identica la cual se utilizará para cifrar y descifrar los datos.

7. Autenticación del cliente: En este punto la conexión ya es segura y cifrada, ahora solo quedaría autenticarse mediante una llave pública o una contraseña.

Si con esta explicación no bastó y quieren profundizar aún más los invito a que cuando se conecten a un servidor SSH utilizen el parametro -v para que puedan ver el flujo de conexión en tiempo real. 


Parte 1   
Se establece la conexión.   

![](/assets/images/create-server-ssh/3.png)

Parte 2 y 3   
Si el servidor no es conocido

![](/assets/images/create-server-ssh/4.png)

Si el servidor es conocido.

![](/assets/images/create-server-ssh/5.png)

Parte 4  

![](/assets/images/create-server-ssh/6.png)

Parte 5

![](/assets/images/create-server-ssh/7.png)

Parte 6

![](/assets/images/create-server-ssh/8.png)

Parte 7

![](/assets/images/create-server-ssh/9.png)


Espero que este post te haya ayudado a comprender de mejor manera el protocolo SSH si hay alguna cosita más que agregar lo haré en un futuro hasta pronto.