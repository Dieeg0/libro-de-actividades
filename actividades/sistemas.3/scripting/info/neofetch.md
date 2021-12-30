
```
Curso       : 202122
Área        : Sistemas operativos, scripting, hardware, información del sistema
Descripción : Script consultar y mostrar la información del Sistema
              Funcionamiento similar a Neofeetch
Requisitos  : GNU/Linux, Ruby
Tiempo      : 5 sesiones
```

# Script: info-neofetch

Mostrar la información hardware y software de nuestro sistema.

# 1. Entrega

* Crear un script ruby llamado **info-neofetch.rb**.
* Crear un informe explicando brevemente la actividad.
* La entrega la realizaremos a través del repositorio Git.
* Al terminar etiquetaremos la entrega con *info-neofetch* (git tag...).

# 2. Crear script

## 2.1 Descripción

* Crear un script *info-neofetch.rb*.
* La función de este script será la de buscar información del sistema (tanto software como hardware) y mostrarla en pantalla.

> AYUDA
>
> * Podemos usar la gema "colorize" para mostrar mensajes con colores.
> * Las gemas se pueden consultar en la web de Rubygems

## 2.2 Tarea del script

Crear un script `info-neofetch.sh` con las siguientes funcionalidades:

| Argumentos           | Acción a realizar |
| -------------------- | ----------------- |
| Sin argumentos, o -h | Se mostrará la ayuda. La ayuda es información de cómo debe usarse el script |
| --network | Se muestra información de red. Como por ejemplo: IP, máscara de red, puerta de enlace, si la IP estática o dinámica. La IP del DNS principal, modelo de la tarjeta de red. Podemos averiguar bastante consultando el fichero "/etc/network/interfaces", la salida del comando "ip a", o el comando "ip route". |
| --memory | Se mostrará información de la cantidad de memoria total del sistema, y cantidad de memoria libre. Podemos averiguar bastante consultando el fichero "/proc/meminfo", o la salida del comando "free". |
| --cpu | Se mostrará información del procesador: modelo, número de núcleos, 32/64 bits, tiempo de encendido (uptime). Podemos averiguar bastante consultando el fichero "/proc/cpuinfo", o el comando "uptime" |
| --disk | Se mostrará información de los discos duros: Tamaño de cada partición y su espacio libre. Podemos averiguar bastante consultando los comandos "fdisk -l", "df -hT". |
| --so | Se mostrará información del sistema operativo instalado. Podemos averiguar bastante con el comando "uname -a".|
| --all | Se mostrará toda la información del sistema. |

* Si el script temina correctamente debe devolver 0 (exit 0).

> AYUDA: Para filtrar la salida proporcionada por los comandos del sistema, podemos usar las siguientes utilidades:
>
> * En ruby tenemos método split().
> * En Bash tenemos muchos comandos. Ver más información en el ANEXO

Comandos útiles:
* head: extrae las primeras líneas de un fichero o salida de datos
* tail: extra las últimas líneas de un fichero o salida de datos.
* wc: contador de líneas o palabras.
* lsusb: Muestra información de los dispositivos usb.
* lspci: Muestra información de los dispositivos PCI.

# 3. Privilegios

* Para ejecutar el script con los privilegios adecuados, configuraremos /etc/sudoers (comando visudo) para poder usar el comando sudo.
* Además el script cuando se ejecute debe comprobar si somos el usuario "root" para poder continuar. En caso contrario finaliza con erro (exit 1).

---

# ANEXO: Información de ayuda.

# Comandos: grep, tr y cut

Podemos usar los comandos grep, tr y cut, para filtrar la salida de diversos comandos, y obtener de esta forma la información deseada.

Por ejemplo, el comando ifconfig, nos proporciona información de red, pero es más de lo que buscamos. Podemos usar grep para filtrar las líneas que buscamos:
```
#ifconfig | grep Difus
Direc. inet:172.16.2.9 Difus.:172.16.255.255 Másc:255.255.0.0
```

Podemos usar el comando tr para reemplazar los espacios por dos puntos:
```
# ifconfig |grep Difus |tr -s ' ' ':'
:Direc.:inet:172.16.2.9:Difus.:172.16.255.255:Másc:255.255.0.0
```

Ahora podemos usar el comando cut para cortar la salida y quedarnos con el dato buscado.
```
# ifconfig |grep Difus |tr -s ' ' ':'|cut -f 4 -d ':'
172.16.2.9
```

Este dato lo podemos guardar en una variable para usarlo más adelante:
```
# DIRIP=`ifconfig |grep Difus |tr -s ' ' ':'|cut -f 4 -d ':'`
# echo $DIRIP
172.16.2.9
```

# Comillas de bash

Hay tres tipos de comillas en shell scripting:

La comilla doble muestra la salida y sustituye cada variable por su valor.
```
# echo "$DIRIP"
172.16.2.9
```

La comilla simple muestra el texto de forma literal.
```
# echo '$DIRIP'
$DIRIP
```

La comilla inclinada (francesa) interpreta el contenido de la variable como la ejecución de un comando.
```
# echo `$DIRIP`
172.16.2.9: orden no encontrada
```

# Kernel actual y el soporte de 64 bits

Para saber si el kernel actual soporta 64 bits basta con escribir en consola:
```
# uname -a.
Linux SOBREMESA 2.6.20-16-generic #2 SMP Wed May 23 00:30:47 UTC 2007 x86_64 GNU/Linux
```

Como se puede ver devuelve que el sistema es x86_64 (osea, de 64 bits), si apareciese solo el valor x86 o un valor de i686, el sistema sería de 32 bits. El procesador es de 64 bits cuando aparece `_64` en la descripción de la arquitectura.

# Procesador y el soporte de 64 bits

Para saber si el hardware soporta 64 bit, abrimos un terminal y ejecutamos: `grep flags /proc/cpuinfo`
Si en el resultado aparece lm, entonces soporta 64 bit; si aparece tm, soporta 32 bit; si aparece rm, soporta 16 bit.

Resumiendo:
* rm : 16 bit
* tm : 32 bit
* lm : 64 bit

Por ejemplo:
```
fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext lm 3dnowext 3dnow up
```

Vemos que soporta 64 bits.

# Disco duro

Para determinar el tamaño en bytes de un bloque de una partición formateada, podemos usar el comando siguiente:
```
# tune2fs -l /dev/sda8 |grep Block
Block count: 18802066
Block size: 4096
Blocks per group: 32768
...
```
