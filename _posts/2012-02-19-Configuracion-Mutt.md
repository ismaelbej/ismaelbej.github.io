---
layout: post
title: Configuración de Mutt
author: Ismael
---

Mi configuración para que Mutt sea más amigable.

Durante algún tiempo estuve buscando un cliente que soporte correo
por IMAP y que pueda manejar carpetas conteniendo varias decenas
de miles de correo. Esta búsqueda no resulto fácil, muchos clientes
dicen soportan esta configuración pero les resulta imposible manejar
una carpeta con tal cantidad de correos.

En realidad para todos salvo para [Mutt](http://www.mutt.org), es un
cliente de correo en modo texto. No es muy intuitivo, pero hay
abundante información en internet como configurarlo. Es un cliente
para sistemas Posix como Linux, pero puede usarse en Windows a
través de Cygwin.

# Configuración

La configuración de mutt puede estar en dos lugares `~/.muttrc` o en `~/.mutt/muttrc`

## Servidor y carpetas

Este ejemplo es para un servidor Dovecot, donde las carpetas pueden
estar en el mismo nivel que Inbox.

```sh
set realname = "Ismael"
set from = "ismael@XXXXXXXX"
set imap_user = "ismael"
set imap_pass = "XXXXXXXXXXXXXXXX"
set folder = "imaps://mail.server"
set spoolfile = "+INBOX"
set record = "+Sent Items"
set postponed = "+Drafts"
```

El símbolo `+` delante de las carpetas se refiere a la carpeta
principal. También suele reemplazarse por `=`, no hay ninguna
diferencia entre usar cualquiera de estos símbolos.

La primer pantalla de Mutt se llama índice, para cambiar a otra carpeta
se puede usar el comando `c?` que presenta una lista de carpetas.

## Mejorando el rendimiento

Tener un cache de los mensajes va a mejorar el rendimiento, pues no
tiene que traer del servidor los mensajes que se bajaron anteriormente.

```sh
set mail_check = 120
set header_cache = ~/.mutt/cache/headers
set message_cachedir = ~/.mutt/cache/bodies
set certificate_file = ~/.mutt/certificates
```

## Navegando por las carpetas

Al iniciar Mutt presenta un índice con los mensajes, un mensaje
se muestra en el paginador. Cuando un línea es demasiado larga el
paginador parte la línea y agrega un _marcador_ al principio de
la línea. Con `set marker = no` deshabilitamos estos marcadores.

El `pager_index_lines` controla cuántas líneas del índice se muestran
en el paginador. Los comandos `set sort` y `set sort_aux` indican 
como presentar los mensajes en el índice. Aqui `threads` indica
que primero los agrupa por threas, `last-date-received` que luego
los ordene por fecha, de más antiguo a más reciente.

Con `ignore` indicamos que líneas de la cabecera del mensaje
no es necesario mostrar.

```sh
set markers = no
set pager_index_lines = 6
set sort = 'threads'
set sort_aux = 'last-date-received'
ignore User-Agent
ignore X-Mailer
```

## Coloreando mensajes

Se pueden configurar los distintos componentes de un mensaje.
Para que sea más fácil leer.

```sh
color attachment brightmagenta default
color error      brightred    default
color hdrdefault red          default
color indicator  brightyellow red
color markers    brightcyan   default
color message    brightcyan   default
color normal     default      default
color quoted     brightblue   default
color search     default      green
color signature  red          default
color status     yellow       blue
color tilde      magenta      default
color tree       magenta      default
```
