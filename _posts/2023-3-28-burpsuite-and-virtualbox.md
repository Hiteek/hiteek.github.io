---
layout: single
title: Cómo interceptar peticiones dentro de una máquina virtual con BurpSuite
date: 2023-01-7
classes: wide
header:
  teaser: /assets/images/notes-about-ssh/teaser.jpg
categories:
  - Virtual Machines
  - Tutoriales de BurpSuite
  - BurpSuite
tags:
  - Web Hacking
  - Intercepción de peticiones
---


¿Alguna vez te has preguntado si es posible interceptar las consultas que realizas con tu navegador dentro de una máquina virtual? ¡Yo también lo hice! Durante una de mis reuniones, un compañero de trabajo me planteó esta pregunta un poco fuera de lo común y, aunque al principio pensé que era innecesario, luego me di cuenta de que puede ser muy útil, especialmente para aquellos que tienen un sistema principal en Windows y una máquina virtual en Linux con pocos recursos que no puede mantener abiertos el navegador y BurpSuite al mismo tiempo.

Así que decidí curiosear un poco y, ¡adivina qué! ¡Encontré la respuesta! En este artículo te enseñaré cómo interceptar fácilmente las consultas dentro de una máquina virtual utilizando BurpSuite. ¡Acompáñame en este sencillo proceso y descubre cómo hacerlo tú también!

# Materiales

- BurpSuite
- Oracle VM VirtualBox
- FoxyProxy Standard
- Alguna distribucion basada en Debian (Ubuntu, Kali, Parrot, ...)

# Empezamos!

## Configuración de la Máquina Virtual

La configuración de la red en la máquina virtual debe tener la siguiente configuración Adaptador Puente

 ![](/assets/images/burpsuite-and-virtualbox/2.png)

PD: Si no encuentra su adaptador de Red puede ser porque los drivers de Red para virtualbox no se instalaron correctamente o tenga un conflicto con VMware, en este caso recomiendo reinstalar VirtualBox.

Esto es para que la máquina virtual se encuentre en la misma red que el host

 ![](/assets/images/burpsuite-and-virtualbox/3.png)

Ahora apuntamos las IP de la máquina host y la máquina virtual que tengan la misma mascara de red.

* Máquina Host: 192.168.1.4

 ![](/assets/images/burpsuite-and-virtualbox/4.png)

* Máquina Virtual: 192.168.1.19

 ![](/assets/images/burpsuite-and-virtualbox/5.png)



## Configuración del Navegador

En mi caso, utilizare firefox en Kali Linux y antes de empezar deberán instalar el certificado CA para que puedan navegar por URLs HTTPS de forma normal.

Aqui la guía oficial de Portswigger: https://portswigger.net/burp/documentation/desktop/external-browser-config/certificate/ca-cert-firefox


Instalamos el FoxyProxy Standard.

 ![](/assets/images/burpsuite-and-virtualbox/1.png)

Añadimos un Proxy con la siguiente configuración de forma que apunte a la máquina host (192.168.1.4).

 ![](/assets/images/burpsuite-and-virtualbox/6.png)

## Configuracion de Burpsuite

En BurpSuite, configuramos el siguiente listener en la ruta Proxy>Intercept>Proxy settings>Proxy listeners

 ![](/assets/images/burpsuite-and-virtualbox/7.png)


Listo ahora puedes interceptar peticiones de un navegador en una máquina virtual desde tu máquina Host.

 ![](/assets/images/burpsuite-and-virtualbox/8.png)


Espero que este post sobre cómo interceptar peticiones de una máquina virtual con BurpSuite en la máquina host haya sido de gran ayuda para ti. ¡Hasta pronto!

