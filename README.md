# Spoofing-y-phising-con-mailserver-
Este proyecto es únicamente con fines educativos y con la intención de ampliar mi portafolio. 

Aqui se documentara una guia detallada de como hacer una SIMULACION  de como se veria un ataque real de spoofing y phishing. Como herramientas para este proyecto se usaron 2 maquinas virtuales, una que sera la del servidor de correos(smtp) para la cual se usaron las sigientes aplicaciones:

Roundcube: Esta es la que nos va a aydar a tener una interfaz de correo electronico con buzon y todo para poder enviar mensaje, esto se uso unicamente por estetica pero facilmente puede funcionar sin esto

Dovecot: Este se va a usar para que se pueda hacer un almacenamiento de correos electronicos en el cual roundcube pueda accer para poder hacer el mailbox del usuario

Postfix: Este es el que se va a encargar del proceos de enrutar, entregar y recibir correos electronicos

Mailx: Este se uso para realizar pruebas antes de roundcube y poder hacer envio por linea de comando

BIND9: Se uso para resolver los nombres de dominio y poder acceder al correo con fakefacebook.com

Dicho ya lo que se va a usar empezamos con el paso a paso:

El proceso realizado fue el siguiente en una máquina virtual de ubuntu server se montaron los siguientes programas postfix como web mail server, mailx para poder hacer envíos y pruebas, dovecot que también es un servicio de webmail pero este nos permite el tema de almacenar y acceder a los correos almacenados por medio de IMAP y POP3 y roundcube el cual se utilizó para poder tener una interfaz gráfica. Además de esto se implementó un servidor DNS con BIND9 para poder usar los dominios “mail.fakefacebook.com” para poder ingresar a la interfaz gráfica desde una VM aparte, cabe recalcar que toda la configuración se hizo localmente por medio de bridge dejare a continuacion las redes usadas:


VM atacante  == Parrot ( 192.168.1.9)
VM smtp == Ubuntu server ( 192.168.1.8)
VM víctima == Ubuntu (192.168.1.10)

Continuando explicare como se configuró cada uno de las aplicaciones

Postfix: La instalación de postfix es la más importante pues este es el servidor de correo pero que es en sí: Postfix nos permite la transferencia de correo la cual su funcionalidad es enrutar y entregar correos en sistemas  Unix y Linux. Se eligió postfix pues este maneja una configuración sencilla, la cual se explicará a continuación.

como primer paso se realizó la instalación de
sudo apt-install postfix 

en este punto se va abrir una interfaz en la cual tendremos que elegir nuestro System mail name este es muy importante pues esto es literal lo que define el @dominiodecorreo.com y este era el mismo que se iba a configurar en el BIND9 dns despues en este caso se uso fakefacebook.com

<img width="551" height="283" alt="{48913169-EBE2-4DAD-915C-7C55A9674EF6}" src="https://github.com/user-attachments/assets/1a4633f2-9b65-4e60-8065-4bc158c1f490" />

ya después de esto se crearon los usuarios de correo en este caso bob y alice para poder hacer los envíos de prueba se usó mailx siendo así que ya quedaria el postfix un ejemplo de uso es este

<img width="568" height="169" alt="{6ACB74DA-EA7C-465D-B22F-262CEA737553}" src="https://github.com/user-attachments/assets/147f75c9-bfd6-4b92-93d8-b0efda9ddd96" />

sin embargo, esto después cambiará pues esa ruta para ver los mails cambia cuando implementamos el dovecot pues al implementar roundcube tenemos que hacer los correos los almacene dovecot para así hacer que roundcube acceda a esto para poder hacer el mailbox siendo así como pasamos a la configuración de dovecot la cual es muy sencilla 

sudo apt-install dovecot-pop3

una vez instalado dovecot no dirigimos a la carpeta nano/etc/postfix/main.cf con el fin de cambiar las siguientes configuraciones

<img width="708" height="275" alt="{2CED6988-349A-429C-A1D6-38B49750B3E7}" src="https://github.com/user-attachments/assets/54cecbd0-513a-4d98-8db4-974055a64d80" />

aquí evidenciamos que se cambió la ruta de guardado del mailbox en postfix para que después roundcube pueda acceder a estos y el cambio en dovecot está en la carpeta nano/etc/dovecot/conf.d/10-mail.conf

<img width="602" height="176" alt="{A34E34BA-876C-408C-85C1-7B0640A187E9}" src="https://github.com/user-attachments/assets/8d3f5973-4e5a-4bc5-a922-ed721816fecc" />

Aquí pasamos al paso mas importante y que mas se dificulto pues es la configuracion del DNS se tuvo dificultades pues la documentación de como hacer este servidor era un poco confusa y la IA no fue de ayuda. Este servidor nos ayudó a que primero pudiéramos ingresar con el dominio mail.fakefacebook.com y que el roundcube pudiera conectarse al postfix para poder hacer el envío de los correos por @fakefacebook.com.

<img width="703" height="389" alt="{2327B393-31A1-4EEB-B422-E849B1C533B9}" src="https://github.com/user-attachments/assets/e3289514-933d-4883-ba85-e4cba45ac2a7" />

En esta configuración las partes importantes son los host principales que son las formas de poder acceder al dominio y la parte de  las estaciones de trabajo que iban a ser la VM que van conectadas al DNS para poder recibir los correos, aquí el nombre de servidor más importante es el mail pues fue el que se configuró después en roundcube para el acceso y la creación de la página con apache

Ya por último y de los pasos más largos pero sencillos pasamos a la instalación de roundcube. Para la instalación de roundcube es indispensable tener MYSQL instalado pues roundcube necesita acceder a una DB para poder acceder a los correos y almacenarlos entonces se instalo mysql y despues se instalo el servicio de imapd de dovecot que también va de la mano con el almacenamiento de correos pero para que las VM puedan acceder a sus mailbox.

Ahora si se procedió con la instalación de roundcube el cual salta una interfaz en el cual tenemos que decirle si tenemos el MYSQL activado y nada más. Después se procedió a hacer la configuracion del apache con roundcube para esto se usó el comando cd/etc/apache2/sites-available/ para encontrar la configuración por defecto de apache se hizo un copy de este archivo con el nombre de round.conf y ahí se hizo la siguiente config

