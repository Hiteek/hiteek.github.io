---
layout: single
title: Happy Android 1
date: 2025-07-2
classes: wide
header:
  teaser: /assets/images/happy-android-I/teaser.jpg
categories:
  - Android
tags:
  - Research
---


Decidí hacer este post porque me interesa mucho el funcionamiento interno de Android y quería plasmar lo que he aprendido en algún lugar, con la esperanza de que también pueda servir como guía o ayuda para otras personas.

# Arquitectura de Android

 ![](/assets/images/happy-android-I/1.png)

A continuación, hablaremos sobre los diversos componentes que forman parte de las distintas capas de la plataforma Android.

## Linux Kernel

En esta capa del sistema operativo se encuentran conceptos fundamentales para el funcionamiento interno de Android, especialmente en lo que respecta a la comunicación entre procesos. Dos de los más importantes son IPC y Binder.

### IPC
La Inter-Process Communication (IPC) es el conjunto de técnicas y mecanismos que permiten que dos o más procesos puedan intercambiar información entre sí, ya sea dentro del mismo dispositivo o a través de una red.
En el contexto de Android, este tipo de comunicación es esencial. Por ejemplo, cuando una aplicación solicita acceso al GPS, en realidad no accede directamente al hardware. En su lugar, se comunica mediante un mecanismo de IPC con un proceso del sistema que sí tiene el control del servicio, y es este quien le devuelve la información solicitada.

### Binder
Binder es el principal mecanismo de IPC en Android. Se trata de una implementación optimizada que permite la comunicación eficiente y segura entre procesos.   

En resumen
* IPC define el "qué" (la comunicación entre procesos)
* Binder es el "cómo" lo hace Android (el mecanismo concreto que lo implementa)

### Androidisms

Hablando de Binder, el sistema de comunicación entre procesos en Android, podemos ir un poco más allá y entrar en un concepto más amplio: las modificaciones que Android realiza sobre el kernel de Linux. Estas adaptaciones específicas se conocen —de manera informal— como “Androidisms”.

Android no es una simple distribución de Linux. Para adaptarlo al entorno móvil, Google introdujo varios cambios que permiten un uso más eficiente de los recursos y una mejor gestión de la energía, la seguridad y la comunicación entre componentes del sistema.

Algunos ejemplos son:

* LMKD (Low Memory Killer Daemon): Se encarga de liberar memoria cerrando procesos menos prioritarios cuando el sistema está bajo presión. Antes era un controlador, ahora opera desde el espacio de usuario.

* Wakelocks: Permiten que una app evite que el dispositivo entre en suspensión.

* Ashmem (Anonymous Shared Memory): Facilita el compartir memoria entre procesos de forma controlada y eficiente.

* Paranoid Networking: Restringe el acceso a sockets de red, asegurando que solo procesos autorizados puedan usar la red.

* Binder: Es el mecanismo principal de IPC en Android, optimizado para permitir una comunicación segura y rápida entre procesos.

## Init

El init de Android es el primer proceso que se ejecuta al arrancar el sistema (PID 1) y se encarga de iniciar y configurar todo el entorno Android, desde montar particiones y aplicar políticas de seguridad hasta lanzar servicios esenciales como Zygote o SurfaceFlinger. A diferencia del init tradicional de Linux (como SysV o systemd), Android utiliza archivos .rc con un lenguaje de configuración propio para definir servicios, sockets, permisos y propiedades del sistema, adaptándose así a las necesidades específicas de un entorno móvil.

## Native Daemons

Cuando usas un dispositivo Android, muchas cosas suceden detrás de escena para que todo funcione como esperas: desde abrir una app hasta conectarte a una red Wi-Fi o insertar una memoria USB. Pero… ¿qué hace que todo eso ocurra automáticamente?

Aquí es donde entran los Native Daemons.
Estos son procesos nativos del sistema operativo que se ejecutan en segundo plano y se encargan de tareas clave. No los ves directamente, pero sin ellos Android simplemente no funcionaría bien.

Veamos tres ejemplos importantes:

* vold: Administra el almacenamiento.

* netd: Controla la red y las conexiones.

* zygote: Lanza las aplicaciones.

## Native Libraries
Las Native Libreries (Librerias nativas) de Android son módulos de código compilados en código máquina para una arquitectura de hardware específica, generalmente escritos en C o C++. Estas bibliotecas son distintas del bytecode de Java o Kotlin que conforma la mayoría de las aplicaciones de Android. Pueden venir por defecto en el sistema operativo y ser usadas por las aplicaciones cuando se requiera o ser compiladas por los desarrolladores para uso exclusivo de un aplicacion.

Ejemplos de librerias nativas del sistema:

* libc: Implementacion personalizada de google de la biblioteca estandar de C diseñada especificamente para el sistema operativo de android.
* libbinder: Implementacion del mecanismo Inter-Process Communicaction en Android.
* liblog: Centraliza logs del sistema y de apps, los envia al logcat.

## HAL

El HAL (Hardware Abstraction Layer) en Android es una capa de abstracción que permite al sistema operativo comunicarse con el hardware sin depender de los detalles específicos de cada dispositivo.

## Maquinas virtuales

* JVM: Es la máquina virtual de Java, diseñada para ejecutar bytecode de Java en cualquier plataforma.

* Dalvik: Es la máquina virtual original de Android, basada en la idea de la JVM, pero adaptada a móviles. Compilador Just In Time, compila en tiempo de ejecucion.
![](/assets/images/happy-android-I/image.png)
* ART: Reemplazo moderno de Dalvik, introducido en Android 5.0+. Compilador  Ahead Of Time, compila durante la instalación.
![](/assets/images/happy-android-I/image-1.png)
Diferencias: Dalvik consume más batería, ART consume más memoria.
 ![](/assets/images/happy-android-I/2.png)

En Android Nougat, ART implementa JIT (Profile-Guided Compilation), lo que permite que los métodos utilizados con mayor frecuencia se almacenen en caché para mejorar el rendimiento. En Android Pie, ART se actualizó a Profiles in the Cloud, donde todos los métodos, clases y funciones que los usuarios suelen usar se recopilan en un archivo llamado Common Core Profile, el cual define qué código precompilar primero durante la instalación de la app. Posteriormente, ART genera un perfil personalizado del usuario, con los métodos que cada usuario utiliza normalmente, para recompilarlos durante el período de carga y optimizar aún más el rendimiento.

## Java Runtime Libraries

Las librerías de Java son bibliotecas que proporcionan las clases y funciones necesarias para que las aplicaciones funcionen. Los archivos DEX de los programas solo contienen referencias a estas clases.

Por otro lado, los APK incluyen varios componentes esenciales: los archivos DEX, los recursos, el AndroidManifest.xml, las librerías nativas y la firma digital que garantiza la integridad y autenticidad de la aplicación.

## System Services
Los System Services en Android son servicios del sistema que proporcionan funcionalidades esenciales

* WindowManagerService: Gestiona las ventanas y la interfaz de usuario, controla la disposición de las apps en pantalla y la interacción con el usuario.

* TelephonyManager: Maneja todo lo relacionado con telefonía, como llamadas, SMS y el estado de la red móvil.

* ConnectivityService: Controla la conectividad de red, incluyendo Wi-Fi, datos móviles y VPNs.

* PowerManager: Gestiona el estado de energía del dispositivo, apagado/encendido de la pantalla, modos de suspensión y ahorro de batería.

* PackageManager: Administra las aplicaciones instaladas, permisos, paquetes y actualizaciones.

Todos estos servicios se comunican mediante Binder y operan bajo permisos estrictos para proteger el sistema.

## Binder

![](/assets/images/happy-android-I/4.png)

En Android, se utiliza la interfaz /dev/binder para permitir la comunicación entre procesos (IPC).

Los procesos no pueden acceder directamente a la memoria de otros procesos; en su lugar, el kernel gestiona la memoria y facilita que los mensajes se envíen entre procesos mediante el controlador Binder.

Algunas implementaciones de Binder a nivel alto incluyen: Intents, ContentProviders, Messenger, entre otros, que permiten a las aplicaciones y servicios comunicarse de forma segura y estructurada.

* Intent: Permite que las aplicaciones soliciten acciones al sistema o a otras apps. Por ejemplo, si se quiere abrir una URL, el sistema puede usar un Intent implícito y seleccionar automáticamente un navegador web disponible.

* ContentProvider: Permite compartir datos de forma segura entre aplicaciones. Por ejemplo, una aplicación de galería de fotos puede usar un ContentProvider para que otras apps, como un editor de imágenes o una red social, accedan a las imágenes del dispositivo sin comprometer la seguridad.

## Bibliotecas especificas de android

* Actividades, servicios y ContentProviders (android.app.*): Proporcionan las clases necesarias para crear actividades, gestionar servicios en segundo plano y exponer datos mediante ContentProviders.

* Interfaz gráfica (android.view.* y android.widget.*): Contienen las clases para construir interfaces de usuario, manejar layouts, widgets y eventos de interacción.

* Acceso a datos (android.database.* y android.content.*): Permiten gestionar bases de datos, archivos, preferencias y comunicación entre componentes mediante Intents y otros mecanismos.

## Aplicaciones

* Apps del sistema: Se encuentran en /system/priv-app. Tienen más privilegios que otras aplicaciones. Están firmadas por el fabricante o el sistema para garantizar su integridad.

* Apps instaladas por el usuario: Se ubican en /data. Cada aplicación opera dentro de su propio sandbox, lo que aísla sus datos y evita que otras apps accedan directamente a su información.

Espero que este post sobre cómo interceptar peticiones de una máquina virtual con BurpSuite en la máquina host haya sido de gran ayuda para ti. ¡Hasta pronto!

## Android Security

El sandboxing en Android es un mecanismo de seguridad que aísla cada aplicación del resto del sistema y de otras apps. Cada aplicación se ejecuta en su propio proceso y bajo un identificador de usuario único (UID), lo que impide que acceda directamente a la memoria o archivos de otras apps. Además, cada app solo tiene los permisos que el usuario le concede, lo que protege los datos, limita el impacto de aplicaciones maliciosas y garantiza que el sistema funcione de manera segura.
![](/assets/images/happy-android-I/5.png)