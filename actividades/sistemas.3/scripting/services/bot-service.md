
```
Curso       : 201920
Area        : Sistemas operativos, scripting, servicios, procesos
Descripción : Crear scripts para crear un servicio.
              Gestionar y configurar servicio para tareas de sysadmin.
Requisitos  : GNU/Linux, Ruby, Telegram.
Tiempo      : 6 sesiones
```

---
# Servicio de comunicaciones con Telegram (bot-service)

Vamos a crear un servicio de Systemd que iniciará un Bot de Telegram.

Propuesta de rúbrica:

| ID  | Criterio | Muy bien(2) | Regular(1) | Poco adecuado(0) |
| --- | -------- | ----------- | ---------- | ---------------- |
| 1.4 | Entrega  ||||
| 2.4 | Entrega  ||||
| 3.2 | Entrega  ||||

---
# 1. Crear un bot de Telegram con ruby

Pasos:
1. Registrar el bot con BotFather
2. Asociar programa ruby con nuestro bot.

# 1.1 Crear bot con BotFather

1. Iniciar Telegram.
1. Buscar usuario `@BotFather`.
1. Escribir el mensaje/orden `/newbot`:
    * Bot name: `@Bot_dvarrui`
    * Username: `dvarrui_bot`

Apuntar los siguientes datos porque los vamos a usar más adelante:
```
Bot URL          : ***
TOKEN (HTTP API) : ***
```

> Usar `/help` para consultar el listado de comandos del bot.
>
> Para una descripción completa del Bot API, consultar la página siguiente: https://core.telegram.org/bots/api

* Crear un grupo de Telegram y añade tu bot.

---

## 1.2 Crear un programa de Ruby

> Enlaces de interés:
> * [Create telegram bot in ruby](https://www.sitepoint.com/quickly-create-a-telegram-bot-in-ruby/)
> * [gem telegram-bot-ruby](https://github.com/atipugin/telegram-bot-ruby)

* `gem install telegram-bot-ruby`, instalar la gems de ruby que permite comunicarse con Telegram.
* Crear un fichero `hide.token` con el valor de nuestro TOKEN.
* Ejecutar este programa de ejemplo [bot-demo.rb](files/bot-demo.rb).

## 1.3 Personalizar el bot

Modificar el `bot-demo.rb` y personalizarlo de la siguiente forma:
* Guardar el bot en `/usr/local/bin/botXXd`.
* Quitar las órdenes "/hello" y "/bye".
* Añadir orden "/help" que muestre una ayuda de los comandos que acepta el bit.
* Añadir orden "/version" que muestre el autor del bot y la fecha de creación.
* Añadir orden "/ip" que muestre la IP del sistema.
* Añadir una orden para que el script ejecute comandos del sistema, y devuelva el resultado al Bot de Telegram.
* Añadir una orden que acepte al menos dos argumentos. Por ejemplo, si quieres que el Bot acepte mensajes con varios argumentos o parámetros... recuerda el uso de `split`. Ejemplo:
```
bot.listen do |message|
 options = message.text.split(" ")
 if options[0] == "/meet"
   # "/meet Mazinger-Z"
   say = "Nice to meet you, #{options[1]}!"
   bot.api.send_message(chat_id: message.chat.id,
     text: say)
 end
end
```

## 1.4 Entrega

* Entregar el script del bot (`/usr/local/bin/botXXd`).
* URL vídeo Youtube donde se muestre el Bot en funcionamiento.

---
# 2. Systemd

> Enlaces de interés:
> * EN - [Use systemd to Start a Linux Service at Boot](https://www.linode.com/docs/quick-answers/linux/start-service-at-boot/)
> * EN - [Systemd System and Service Manager](https://www.freedesktop.org/wiki/Software/systemd/)
> * EN - [Systemd - Unit file](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)
> * EN - [Understanding Systemd Units and Unit Files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)

Vamos a crear un servicio para nuestro Bot, de modo que se inicie siempre al arrancar el equipo y que podemos gestionarlo como el resto de servicios (usando el comando `systemctl`).

## 2.1 Crear un servicio

* Vamos a usar el script demonio, creado en el apartado anterior. Un demonio (daemon) es un programa que se ejecuta en segundo plano permanentemente para dar algún servicio.
* Copiar el script a `/usr/local/bin/botXXd` y hacerlo ejecutable.
* Crear el directorio `/etc/botXX`. Lo usaremos para poner configuraciones de nuestro servicio.
* Crear el fichero `/etc/botXX/token`. Lo usaremos para guardar dentro el TOKEN de nuestro bot de Telegram.
* Modificar el contenido del script para cambiar la ruta del TOKEN:
```
token = %x[cat /etc/botXX/token].strip
```

El init es el proceso que inicia todo el sistema y arranca los servicios. Cada sistema operativo puede tener distintos "init" como Systemd, SystemV, Upstart, Openrc, etc. Nuestro sistema operativo viene con Systemd, así vamos a configurar Systemd para gestionar nuestro servicio.

Cada servicio de Systemd se define en un fichero `Unit file`

* Crear el fichero `/etc/systemd/system/botXX.service` con el siguiente contenido:

```
[Unit]
Description=Servicio Bot del alumnoXX.
After=network.service

[Service]
Type=simple
ExecStart=/usr/bin/ruby /usr/local/bin/botXXd

[Install]
WantedBy=multi-user.target
```

> Así se define un servicio sencillo.
> * `ExecStart`, especifica el comando/demonio que se ejecutará para iniciar el servicio.
> * `After`, indica que este servicio debe iniciarse después del servicio que activa la red.

# 2.2 Iniciar y activar el servicio

* Iniciar sesión como superusuario.
* `systemctl status botXX`, comprobamos que el estado está parado.
* `systemctl start botXX`, para iniciar el servicio.
* `systemctl status botXX`, comprobamos que ahora sí está corriendo.
* `systemctl enable botXX`, para activar el servicio. De este modo se iniciará automáticamente al arrancar el sistema.
* Reiniciamos la MV.
* `systemctl status botXX`, para confirmar que el servicio se ha iniciado automáticamente.

## 2.3 Parar el proceso

Lo más cómodo para parar el servicio es `systemctl stop botXX`, pero también podemos parar el servicio botXX "matando" el programa botXXd. Veamos como:
* `ps -ef | grep botXXd`, localizar el identificador del proceso(PID) botXXd.
* `kill -9 PID`, emitimos una señal/orden (-9) al proceso (PID). La señal -9 ordena al proceso que "muera" (que se cierre).

## 2.4 Entrega

* Fichero de configuración del servicio `/etc/systemd/system/botXX.service`.
* Capturas de pantalla donde se muestre que podemos iniciar y parar el servicio `botXX` con el comando `systemctl`.

---
# 3. Programar tareas

Ya sabemos como configurar nuestro programa demonio (botXXd) para que se comporte como un servicio, y además para que se inicie de forma automática al iniciar la máquina.

Supongamos que nos preocupa que nuestro programa (botXXd) se pueda detener de forma inesperada y no nos demos cuenta. Vamos a crear otro script que se va a encargar de iniciar el demonio si se para. Este script va a controlar el demonio cada 5 minutos.

## 3.1 Controlador

Creamos un nuevo script `/usr/local/bin/botXXcontroller`. Este script hará lo siguiente:
* Consulta si el demonio está en ejecución. Se puede hacer con:
    * `ps -ef |grep botXXd` o
    * `systemctl is-active botXX` o
    * `systemctl status botXX`
    * etc.
* Si el servicio no está en ejecución, entonces:
    * Se inicia el servicio (`systemctl ...`).
    * Se registra el siguiente mensaje en el fichero de log (`/etc/botXX/log`): "botXXcontroller [Fecha/hora] => Iniciando el servicio botXX..."
* Si el servico está en ejecución, entonces registramos el siguiente mensaje en el fichero de log (`/etc/botXX/log`): "botXXcontroller [Fecha/hora] => No hace nada!".
* Se termina el script.

## 3.2 Tareas programadas

> Enlaces de interés:
> * [Cómo utilizar crontab para programar tareas](https://www.redeszone.net/2017/01/09/utilizar-cron-crontab-linux-programar-tareas/)

Vamos a programar el script `botXXcontroller` para que se ejecute cada 5 minutos. Usaremos la herramienta crontab.

* Iniciar sesión con el usuario `root`.
* `crontab -l`, vemos que no hay ninguna configuración creada.
* `crontab -e`, se nos abre un editor.
* Pulsar `i`(insert) para activar el modo de empezar a escribir.

> Información para configurar crontab:
> * m: minuto
> * h: hora
> * mon: mes
> * dow: día del mes
> * dom: día de la semana (0=domingo, 1=lunes, etc)
> * Comando a ejecutar

* Escribir algo parecido a lo siguiente:
```
*/5 * * * *   /usr/local/bin/botXXcontroller
```

Esta configuración programa una ejecución del script cada 5 minutos.

* Cuando terminemos, pulsar `ESC`.
* Escribir `:`, `wq`. Así grabamos(w=write) y salimos (q=quiet) del editor de crontab.
* `crontab -l`, para consultar la tarea programada.
* Paramos el servicio.
* Esperamos 5 minutos y ahora debe haberse iniciado de forma automática (con crontab).

## 3.3 Entrega

* `botXXcontroller`.
* Configuración `crontab -l`.
* Captura de pantalla con la siguiente secuencia:
    1. `systemctl status botXX`, mostrando el servicio parado.
    1. Esperar 5 minutos.
    1. `systemctl status botXX`, mostrando el servicio activo.
* Contenido del fichero `/etc/botXX/bot.log`.

---
# 4. Preguntas

* _¿Se podría instalar Ruby en Windows?_ => RubyInstaller
* _¿Se podría pasar el demonio (botXXd) a Windows?_ => Cambiando los comandos de GNU/Linux dentro del script por los comandos de Windows.
* _¿Sabrías crear una tarea programa en Windows?_ => Ir a `Panel de control -> tareas programadas`.

# ANEXO

https://github.com/J-Rios/TLG_JoinCaptchaBot

GitHub (https://github.com/J-Rios/TLG_JoinCaptchaBot)
GitHub - J-Rios/TLG_JoinCaptchaBot: Telegram Bot to verify if users joining a group are human. The Bot sends an image captcha for each new user and kicks any of them who can't solve the captcha in a specified time.
Telegram Bot to verify if users joining a group are human. The Bot sends an image captcha for each new user and kicks any of them 

