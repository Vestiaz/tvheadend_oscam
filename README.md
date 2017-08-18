# tvheadend_oscam
Guía Básica de configuración de tvheadend junto con oscam

¿Qué se necesita?

-Una sintonizadora de satelite (yo tengo una DVBSky S960) 

-Un equipo en el que se tenga instalado linux donde instalaremos y configuraremos los programas (pc, raspberry, etc)

-Parabolica individual o comunitaria, da lo mismo a efectos prácticos para sintonizar con ASTRA 19,2º


Hay que instalar tvheadend, para lo que hay que añadir el repositorio adecuado según nuestro hardware/software.

Aquí la info para añadir el repositorio: [url]https://tvheadend.org/projects/tvheadend/wiki/AptRepository[/url]

Primero hay que añadir esta clave:

[CODE]sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61 
[/CODE]Y a continuación se añade el repositorio donde "DISTRO BUILDTYPE " variará en función del sistema que tengamos:

[CODE]echo "deb https://dl.bintray.com/tvheadend/deb DISTRO BUILDTYPE" | sudo tee -a /etc/apt/sources.list[/CODE]Por ejemplo, para ubuntu 16.04 xenial usaríamos:

[CODE]echo "deb https://dl.bintray.com/tvheadend/deb xenial stable" | sudo tee -a /etc/apt/sources.list[/CODE]El caso de la raspberry es un poco especial y hay que usar estas otras instrucciones: [url]https://tvheadend.org/boards/5/topics/21528?r=23476[/url]

En mi caso uso la distribución unstable porque así tvheadend tiene más opciones.

Una vez añadida la clave y el repositorio hay que actualizar con el comando:

[CODE]sudo apt-get update[/CODE]Y después instalar tvheadend:

[CODE]sudo apt-get install tvheadend[/CODE]Si da error, hemos de revisar si hemos añadido el repositorio correctamente.

Durante la instalación nos pedirá un usuario administrador y su contraseña. Será necesario para podernos loguear en tvheadend.

Una vez instalado, para configurarlo hay que acceder con un navegador a la dirección: [url]http://ipdelequipo:9981[/url]

La verdad es que el programa se las trae porque tiene multitud de opciones y configuraciones, yo me voy a quedar en lo básico.

Según la versión de tvheadend que hayamos instalado (por eso recomiendo el repositorio unstable) al entrar se ejecuta un asistente para su configuración.

Nos pide el lenguaje a usar (spanish), el usuario administrador y su contraseña, después busca los sintonizadores que tengamos y en tuner escogemos nuestro sintonizador (si está conectado y no aparece habrá que lidiar con los drivers correspondientes) En network type establecemos la red que nos interesa (dvb-s para el satélite) y en network name establecemos un nombre cualquiera por ejemplo satelite. En los presintonizados escogemos ASTRA 19,2º

Entonces se pone a escanear e irán apareciendo muxes y servicios, que después mapearemos a los canales en la siguiente pantalla del asistente.

Una vez finalizado el asistenete, ya deberíamos ver en configuration-Channel/EPG la lista de canales.

Vamos a crear una CA para que tvheadend se pueda comunicar con oscam que es el programa que se encargará de desencriptar los canales.

Para eso vamos a Configuration-CAs y añadimos una con los siguientes valores:

Enabled (sí)
mode: OSCam pc-nodmx (rev >= 9756)
Camd.socket filename / IP Address (TCP mode): /tmp/camd.socket
Listen / Connect port: 0

Y guardamos.

Ahora la CA estará en rojo, es lógico porque todavía no hemos instalado oscam. Cuando todo esté correcto la mostrará en verde.

Vamos a instalar ahora oscam con este gran tutorial: [url]https://tvheadend.org/boards/13/topics/6211[/url]

Hay que tener en cuenta que las siguientes operaciones hay que hacerlas como root, así que o bien ponemos sudo delante de cada uno de los comandos o bien antes de empezar hacemos "sudo su" para ser root.

[CODE]Instalar herramientas:
    apt-get install apt-utils dialog usbutils gcc g++ wget build-essential subversion libpcsclite1 libpcsclite-dev libssl-dev cmake make libusb-1.0-0-dev nano

Nos bajamos las fuentes
    cd /usr/src 
    svn checkout http://www.streamboard.tv/svn/oscam/trunk oscam-svn

Configuración, construcción e instalación de oscam
    cd oscam-svn
Configurar opciones
    make config
Compilar Oscam
    make OSCAM_BIN=./build/oscam
Copiar Oscam a su destino y establecer permisos
    cp /usr/src/oscam-svn/build/oscam /var/local/oscam
    chmod 755 /var/local/oscam

Copiamos el fichero de configuración de ejemplo 
     cp /usr/src/oscam-svn/Distribution/doc/example/oscam.conf /usr/local/etc/

Editamos el fichero de configuración y habilitamos webif (servidor web)
    nano /usr/local/etc/oscam.conf 

Configuración de ejemplo:

# oscam.conf generated automatically by Streamboard OSCAM 1.20-unstable_svn SVN r11213
# Read more: http://www.streamboard.tv/svn/oscam/trunk/Distribution/doc/txt/oscam.conf.txt

[global]
logfile                       = /var/log/oscam/oscam.log
nice                          = -1
usrfile                       = /var/log/oscam/oscamuser.log
cwlogdir                      = /var/log/oscam/cw

[anticasc]
enabled                       = 1
numusers                      = 1
samples                       = 5
penalty                       = 1
aclogfile                     = /var/log/oscam/aclog.log
denysamples                   = 4

[cache]

[cs378x]
port                          = 30000@0100;30001@0200:FFF000,FFFF00;30002@0300

[newcamd]
port                          = 10000@0100:FFFFFF;10001@0200:FFF000,FFFF00;10002@0300:FFFFFF
key                           = 000102030405060708090A0B0C0D

[radegast]
port                          = 20000
allowed                       = 192.168.0.0-192.168.255.255
user                          = user1

[serial]
device                        = user2@/dev/ttyS0?delay=1&timeout=300;user3@192.160.0.10,2006?delay=1&timeout=5000

[gbox]
port                          = 50000
hostname                      = host.example.com

[cccam]
port                          = 40000
nodeid                        = 25164155109D13B2
version                       = 1.2.3
reshare                       = 2

[dvbapi]
enabled                       = 1
au                            = 1
pmt_mode                      = 0
user                          = tvheadend
boxtype = pc-nodmx

[monitor]
port                          = 988
aulow                         = 120
monlevel                      = 1

[webif]
httpport                      = 8888
[COLOR=red]httpuser                      = admin
httppwd                       = contraseña[/COLOR]
[COLOR=red]httpallowed                   = 0.0.0.0-255.255.255.255
[/COLOR]
####################

Asegurate de cumplimentar los campos marcados en rojo. usuario, contraseña y redes permitidas para acceder al webif.

Para que oscam se inicie en el arranque creamos un script de inicio
    nano /etc/init.d/oscam

añadir todo lo contenido entre las dos lineas ============

=================
#! /bin/sh
### BEGIN INIT INFO
# Provides:          Oscam
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Oscam init script
# Description:       Launch oscam at startup
### END INIT INFO

DAEMON=/var/local/oscam
DEAMON_OPTS="-b -r 2" 
PIDFILE=/var/run/oscam.pid

test -x ${DAEMON} || exit 0

. /lib/lsb/init-functions

case "$1" in
    start)
log_daemon_msg "Starting OScam" 
start-stop-daemon --start --quiet –user- --background --pidfile ${PIDFILE} --make-pidfile --exec ${DAEMON} -- ${DAEMON_OPTS}
log_end_msg $?
    ;;
    stop)
log_daemon_msg "Stopping OScam" 
start-stop-daemon --stop --exec ${DAEMON}
log_end_msg $?
    ;;
    force-reload|restart)
    $0 stop
    $0 start
    ;;
  *)
    echo "Usage: /etc/init.d/oscam {start|stop|restart|force-reload}" 
    exit 1
    ;;
esac

exit 0

=================

Establecemos los permisos
    chmod 755 /etc/init.d/oscam

Hacemos que el script de inicio se ejecute en el arranque
    update-rc.d oscam defaults

Creamos ficheros y directorios para oscam y establecemos sus permisos
    mkdir /var/log/oscam
    touch /var/log/oscam/oscam.log
    chmod 755 /var/log/oscam/oscam.log
    touch /var/log/oscam/oscamuser.log
    chmod 755 /var/log/oscam/oscamuser.log
    mkdir /var/log/oscam/cw
    chmod 755 /var/log/oscam/cw

Iniciar oscam
    /etc/init.d/oscam start[/CODE]Vamos a acabar de configurar oscam. Accedemos a [url]http://ip:8888[/url]

Vamos a files-oscam.user y pegamos esto:

[CODE][account]
user                          = tvheadend
pwd                           = contraseña
au                            = 1
group                         = 1[/CODE]Y le damos a save para guardarlo.

Ahora hay que poner las clines cccam en los readers. Vamos a files-oscam.server y pegamos lo siguiente:

[CODE][reader]
label=line1
enable=1
protocol=cccam
[COLOR=red]device=direccion de la cline,puerto
user=usuario de la cline
password=contraseña de la cline[/COLOR]
cccversion=2.1.2
group=1
inactivitytimeout=1
reconnecttimeout=30
lb_weight=100
cccmaxhops=10
ccckeepalive=1
cccwantemu=0
 
[reader]
label=line2
enable=1
protocol=cccam
[COLOR=red]device=direccion de la cline,puerto
user=usuario de la cline
password=contraseña de la cline[/COLOR]
cccversion=2.1.2
group=1
inactivitytimeout=1
reconnecttimeout=30
lb_weight=100
cccmaxhops=10
ccckeepalive=1
 
[reader]
label=line3
enable=1
protocol=cccam
[COLOR=Red]device=direccion de la cline,puerto
user=usuario de la cline
password=contraseña de la cline[/COLOR]
cccversion=2.1.2
group=1
inactivitytimeout=1
reconnecttimeout=30
lb_weight=100
cccmaxhops=10
ccckeepalive=1[/CODE]Evidentemente hay que rellenar los campos en rojo con las cccam que se tengan.

Se guarda con save y se recargan los readers yendo a Readers-reload readers.

Si todo ha ido bien, podremos ver que la CA de tvheadend está en verde con lo que ya nos debería de funcionar.

Para ver si hay algún problema interesa ver la consola de tvheadend que se activa en un botón que hay abajo a la derecha de la interfaz de tvheadend.

NOTA: en mi caso, con la sintonizadora USB DVBSky S960 da un problema con los drivers, para solucionarlo, hay que descargar [URL="https://github.com/OpenELEC/dvb-firmware/raw/master/firmware/dvb-demod-m88ds3103.fw"]este fichero[/URL] y copiarlo en /lib/firmware con permisos de root. Se reinicia el sistema y entonces el sintonizador funciona bien. 
Visto aquí: [url]https://tvheadend.org/boards/5/topics/16013[/url]
