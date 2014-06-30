<!--
.. title: Mi experiencia con Tripwire
.. slug: tripwire
.. date: 2014/06/30 12:00:00
.. tags: tripwire, raspberry pi, raspberry, ARM, security, seguridad
.. link:
.. description: Informe sobre la herramienta Tripwire.
.. type: text
-->

¿Qué es Tripwire?
=================

#### Es una herramienta diseñada para sistemas Unix cuya finalidad es la de detectar si ciertos archivos de nuestro sistema han sido modificados sin nuestro permiso.

Es importante resaltar que esta herramienta no reemplaza los sistemas de
bloqueo o detección de intrusión tales como firewalls o IDS’s, si no
que, asumiendo que estos han fallado, revisa nuestro sistema de ficheros
en busca de indicios de una posible intrusión.

¿Cómo funciona?
===============

Mantiene una base de datos cifrada con el hash[^1] de cada archivo
especificado previamente en su configuración.

De esta forma, cuando posteriormente se ejecute la revisión del sistema,
volverá a calcular dichos hash y los comprobará con los almacenados en
la base de datos. Si el hash es igual el archivo no ha sido modificado,
mientras que si el hash cambia es tarea del administrador investigar si
el archivo ha sido modificado debido al natural funcionamiento y
actualización del sistema o a una intrusión.

Nuestro entorno
===============

Para llevar a cabo este informe se ha utilizado un sistema Raspbian[^2]
(Debian Wheezy) instalado en una máquina Raspberry Pi Rev2[^3]
funcionando a 700 MHz con 512MB de RAM.

Instalación
===========

El paquete de instalación se encuentra en el repositorio por defecto de
Debian, por lo tanto el primer paso es ejecutar

\#apt-get install tripwire

Tripwire trabaja con dos claves imprescindibles para su correcto
funcionamiento:

-   La “site key”. Usada para cifrar el fichero de configuración y el de
    políticas que explicaremos en el apartado de Configuración.

-   La “local key”. Usada para cifrar la base de datos y, opcionalmente,
    los archivos de reportes.

En la fase de preconfiguración de la instalación del paquete nos
preguntará si queremos definir en ese momento las dos claves, nosotros
responderemos que sí pero también podríamos definirlas posteriormente.

Una vez finalizada la instalación tendremos que configurar la
herramienta.

Configuración
=============

Existen dos archivos de configuración de Tripwire:

-   El archivo de configuración general: “twcfg.txt”.

-   El archivo de configuración de la política por defecto: “twpol.txt”.

Para configurar Tripwire trabajaremos con estos archivos para, cuando
estén correctamente configurados, generar sus respectivos archivos
cifrados (“tw.cfg”, “tw.pol”), que serán los que use Tripwire para su
funcionamiento.

Archivo de configuración general
--------------------------------

En este archivo se definen parámetros globales para el funcionamiento de
la herramienta, entre ellos se encuentran los siguientes:

-   POLFILE -\> Ruta al archivo de políticas ya cifrado.

-   DBFILE -\> Ruta al directorio que contiene las bases de datos (si
    tenemos varios hosts).

-   REPORTFILE -\> Ruta all directorio donde se almacenarán los archivos
    de reportes.

-   SITEKEYFILE -\> Ruta al archivo que contiene la “site key”.

-   LOCALKEYFILE -\> Ruta al archivo que contiene la “local key”.

-   EDITOR -\> Representa el editor de texto usado dentro de la
    aplicación.

Archivo de políticas
--------------------

Nos encontramos con un archivo de texto muy grande, existen múltiples
parámetros que se pueden modificar, pero nosotros enfocaremos nuestro
trabajo en aprender a definir qué archivos revisará Tripwire, el tipo de
los mismos y los niveles de severidad de las reglas que los agrupan.

### Niveles de severidad

Tripwire permite definir conjuntos de archivos (reglas) que comparten un
“nombre de regla” y un nivel de severidad.

Por defecto, en nuestro archivo de configuración, tendremos los
siguientes niveles de severidad:

-   SIG\_LOW = 33 ;

-   SIG\_MED = 66 ;

-   SIG\_HI = 100 ;

Como podemos observar definen un entero (del 0 al 100) que representa el
nivel de severidad que supondría que algún archivo definido dentro de la
regla de nombre “rulename” fuese modificado.

### Tipos de archivos

Dentro de la definición de una regla, Tripwire permite definir el tipo
de cada archivo a revisar para definir la naturaleza de cambio del
mismo.

Por defecto, en nuestro archivo de configuración, tendremos los
siguientes tipos:

-   SEC\_LOG = \$(Growing) ;

    -   Representan los archivos cuyos permisos o propietario nunca
        deberían cambiar pero que crecen.

-   SEC\_BIN = \$(ReadOnly) ;

    -   Representan los binarios que nunca deberían cambiar.

-   SEC\_CONFIG = \$(Dynamic) ;

    -   Representan los directorios o archivos que son cambiados de
        forma esporádica.

-   SEC\_CRIT = \$(IgnoreNone)-SHa ;

    -   Representan los archivos críticos que nunca deberían cambiar.

-   SEC\_INVARIANT = +tpug ;

    -   Representan los directorios cuyos permisos o propietario nunca
        deberían cambiar.

### Sintaxis

Para configurar este archivo debemos entender primero su sintaxis con un
ejemplo de definición de regla.

\#

\# Critical executables 

\# 

( 

rulename = ‘"Root file-system executables‘", 

severity = \$(SIG\_HI) 

) 

{ 

/bin -\> \$(SEC\_BIN) ; 

/sbin -\> \$(SEC\_BIN) ; 

} 

Podemos observar 3 partes diferenciadas.

-   La primera parte es un breve comentario que describe la regla.

-   La segunda parte está escrita entre paréntesis y define el nombre de
    la regla (“rulename”) y el nivel de severidad (“severity”).

-   La tercera parte está escrita entre corchetes y define una lista de
    directorios o archivos junto con sus respectivos tipos.

Instalación del archivo de políticas
====================================

Una vez configurado el archivo de políticas debemos generar el archivo
cifrado con el que trabajará Tripwire. Usaremos el comando:

\# twadmin -m P /etc/tripwire/twpol.txt

Generación de la base de datos
==============================

Una vez instalado el archivo de políticas debemos general la base de
datos con el comando:

\#tripwire -m i 2 \> /tmp/erroresTripwire

Este comando también escribirá en el archivo /tmp/erroresTripwire una
lista de los errores con los que se encontró la aplicación a la hora de
generar la base de datos.

Es necesario seguir el siguiente procedimiento hasta que no haya errores
en dicho archivo:

1.  Generar la base de datos con el comando antes mencionado.

2.  Revisar el fichero /tmp/erroresTripwire en busca de errores.

3.  Corregir los errores en el archivo de políticas.

4.  Instalar de nuevo el archivo de políticas.

5.  Volver al paso 1.

Una vez se haya generado la base de datos sin errores se deben borrar
los archivos de configuración “twcfg.txt” y “twpol.txt”, habiendo
copiado los mismos previamente a un medio seguro, fuera del entorno de
nuestra máquina. Esto es muy importante ya que, si estos archivos
estubiesen accesibles en nuestra máquina, un atacante podría leerlos y
aprovecharse de esta información para evitar su detección.

Revisión del sistema
====================

Para ejecutar la revisión del sistema basta ejecutar el comando:

\#tripwire -m c \> /tmp/informeTripwire

Escribirá un informe en el archivo /tmp/informeTripwire con los
resultados de la revisión.

Este paso se llevará a cabo cada vez que el administrador del sistema lo
considere oportuno. Debemos tener en cuenta que supone una cierta carga
para máquinas de bajo rendimiento como la usada en nuestro entorno por
lo tanto se aconseja ejecutar las revisiones en horarios de baja demanda
del sistema.

1 http://www.angelcarrasco.com/tag/tripwire/

http://www.escomposlinux.org/lfs-es/blfs-es-5.0/postlfs/tripwire.html

http://linux.die.net/man/8/tripwire

Licencia del documento:
========================

#### Este obra cuyo autor es Pablo López viqueira está sujeta a la licencia Reconocimiento-CompartirIgual 4.0 Internacional de Creative Commons. Para ver una copia de esta licencia, visite <http://creativecommons.org/licenses/by-sa/4.0/deed.es_ES.>

[^1]: En este caso la función hash utilizada es MD5.

[^2]: http://es.wikipedia.org/wiki/Raspbian

[^3]: http://es.wikipedia.org/wiki/Raspberry\_Pi
