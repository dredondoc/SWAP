# Práctica 4: Asegurar la granja web

*El objetivo de esta práctica es configurar todos los aspectos relativos a la seguridad de
la granja web ya creada.
Hay que llevar a cabo las siguientes tareas obligatorias:*
1. *Crear e instalar en la máquina 1 un certificado SSL autofirmado para configurar
el acceso HTTPS a los servidores. Una vez configurada la máquina 1, se debe
copiar al resto de máquinas servidoras y al balanceador de carga. Se debe
configurar nginx adecuadamente para aceptar y balancear correctamente tanto
el tráfico HTTP como el HTTPS.*
2. *Configurar las reglas del cortafuegos con IPTABLES para asegurar el acceso a
uno de los servidores web, permitiendo el acceso por los puertos de HTTP y
HTTPS a dicho servidor. Esta configuración se hará en una de las máquinas
servidoras finales (p.ej. en la máquina 1), y se debe poner en un script con las
reglas del cortafuegos que se ejecute en el arranque del sistema (según la
versión de Linux, se llevará a cabo de una forma u otra).*
*Adicionalmente, y como primera tarea opcional para conseguir una mayor nota en esta
práctica, se propone realizar la instalación de un certificado del proyecto Certbot en
lugar de uno autofirmado. Es importante tener en cuenta que para obtener este tipo de
certificado, es necesario disponer de un dominio real con IP pública (no se puede
hacer en máquinas virtuales).
Como segunda tarea opcional para conseguir una mayor nota en esta práctica, se
propone realizar la configuración del cortafuegos en una cuarta máquina (M4) que se
situará delante del balanceador. Esa M4 sólo tendrá configuradas las iptables, para
hacer el filtrado y posterior reencaminamiento del tráfico hacia el balanceador. En esta
configuración más compleja sólo a esa M4-cortafuegos se le hará la configuración de
iptables (el resto de máquinas de la granja tendrá la configuración por defecto,
aceptando todo el tráfico como política por defecto).
Como resultado de la práctica 4 se mostrará al profesor el funcionamiento del acceso
por HTTPS a las páginas web almacenadas en los servidores finales, haciendo
peticiones con la herramienta curl por HTTP/HTTPS tanto a la máquina 1 como al
balanceador de carga. También se mostrará el funcionamiento del filtrado del tráfico
HTTP/HTTPS con las reglas de iptables configuradas. En el documento de texto a
entregar se describirá cómo se han realizado las diferentes configuraciones (tanto
configuraciones y comandos de terminal a ejecutar en cada momento).*

----

## Acceso por HTTPS

### Generar e instalar un certificado autofirmado

El primer paso es generar un certificado autofirmado en Ubuntu Server. Para ello deben ejecutarse las siguiente orden.

`mkdir /etc/apache2/ssl`

`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout
/etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt`

![](./img/ssl.png)

A continuación debe activarse SSL en Apache. Por lo que empleamos las siguientes ordenes:

`a2enmod ssl`

`service apache2 restart`

Editamos el archivo de configuración de SSL del sitio:

`nano /etc/apache2/sites-available/default-ssl.conf`

Y añadimos las siguientes líneas: 

```
SSLCertificateFile /etc/apache2/ssl/apache.crt
SSLCertificateKeyFile /etc/apache2/ssl/apache.key
```
![](./img/def-ssl.png)

Activamos el sito default-ssl y reiniciamos apache:

`a2ensite default-ssl`

`service apache2 reload`

Para comprobar la correcta instalación, haremos una **petición curl HTTPS**

`curl -k https://<ip>`

En nuestro escenario:
* El equipo 192.168.56.11 tiene HTTPS activo
* El equipo 192.168.56.10 no tiene HTTPS activo

![](./img/https.png)

![](./img/http.png)

Para solucionarlo se debe copiar la clave generada anteriormente al otro servidor y activar SSL.

La copia de las claves se realiza mediante rsync. En la máquina que desea recibir los certificados ejecutamos:

`rsync -avz -e ssh <ip>:/etc/apache2/ssl/* /etc/apache2/ssl/`

![](./img/rsync_crt.png)

Repetimos el proceso con el archivo */etc/apache2/sites-available/default-ssl.conf* 

`rsync -avz -e ssh <ip>:/etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/`

Activamos SSL como se indicaba anteriormente y comprobamos que funciona correctamente.

![](./img/activar-ssl.png)

![](./img/prueba2.png)



----
[Práctica 5](../practica5/practica5.md)