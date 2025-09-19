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

<img width="594" height="402" alt="{C217460D-F436-449E-BC04-A440430AFB96}" src="https://github.com/user-attachments/assets/d103a7a8-9cd1-4ad7-a592-db12d4fb3bdf" />

Aquí se asignó el nombre de acceso del servidor el cual es mail.fakefacebook.com y en la parte final para dar acceso total de privilegios que es una solución de error de permisos. Después en el a2ensite round conf se inicializó para crear el apache y después para crear el enlace simbólico y después agregarlo al DNS bind9 como ya se mostró.

Como último paso de esto se configuro el archivo nano /etc/roundcube/config.inc.ph con el fin de establecer la conexión de roundcube con el postfix

<img width="718" height="486" alt="{E5EA6ED3-B345-49EC-A466-BD40F95D1E5A}" src="https://github.com/user-attachments/assets/5f4a464a-8fd0-4519-b6a5-80fc8cf50222" />

aquí se cambiaron varias cosas pues roundcube ha tenido problemas con este archivo y las versiones entonces se tuvo que cambiar el de autenticación que usaba para acceder al smtp y poner el host que es fakefacebook:25, puesto 25 pues este es el usa por defecto postfix, algo para aclarar es que este puerto va a variar según el protocolo que se use como se usa el por defecto se usa el 25 pero si digamos fuera ssl se tiene que usar el 465. con esto terminaría la configuración del roundcube a continuación una imagen del resultado.

<img width="705" height="328" alt="{8D73DE7D-49C4-42F8-858A-2EDBC6483C6B}" src="https://github.com/user-attachments/assets/f4aadf59-3fb1-49f1-8ff5-5c72885e685e" />
Ahora pasamos a la parte de campaña blanca y la implementación del SET de facebook, se decidió unificar estos dos puntos con el fin de simular un ataque real, cabe recalcar que todo se hizo en un ambiente local.

Esta parte fue la más sencilla pues el sistema operativo ya sea PARROT o KALI  tiene la aplicacion SET (social engineer toolkit), la cual es una aplicación que tiene muchas herramientas para ataques de ingeniería social en este caso suplantación para esto simplemente se abre la app y se despliega el siguiente menú.

<img width="398" height="326" alt="{301FE28D-5E3C-426B-A567-8A8053E32ACF}" src="https://github.com/user-attachments/assets/79e1ca00-0d8b-4f94-99c3-98b209ec8d11" />

Se selecciona el número 2 y se desplegará esto

<img width="378" height="201" alt="{06821387-9190-4140-9F1C-C309F30C7962}" src="https://github.com/user-attachments/assets/bd01f2eb-bae7-4ba8-9edb-3ab65c68903f" />

Donde se elige el 3 y ya solo es cuestión de seleccionar site cloner y rellenar

<img width="606" height="236" alt="{07113FCB-9B28-45D1-8CC9-D29A5F286B42}" src="https://github.com/user-attachments/assets/fea8bd7b-34a4-413b-acba-ac797deb9022" />

la forma de acceso es por la ip de la VM en este caso 192.168.1.9, para poder implementarlo a la campaña de correo balnco esta ip es importante pues será el botón de redirección en el correo con el HTML configurado.

<img width="604" height="200" alt="{9B9902E6-9D90-4CAE-ABC4-ABC30BAC1B04}" src="https://github.com/user-attachments/assets/e887268d-1149-472b-9c32-0b6825a2d2c8" />

el cual este ya después se hizo el envío respectivo

<img width="604" height="283" alt="{A6727766-654C-4E02-A8E7-DB39F40D6561}" src="https://github.com/user-attachments/assets/68090e0c-d165-4118-a4ee-becf6741a567" />

viéndose así en el envío

<img width="603" height="381" alt="{9EB23276-99E5-4DD5-A01E-E5AC28D42E9F}" src="https://github.com/user-attachments/assets/30b6baeb-bbc6-4ab8-bf55-52766f7c8af0" />

Así se ve el envío en roundcube pero también se puede enviar de la siguiente forma por línea de código.

<img width="604" height="25" alt="{4D1AFCA0-4FC6-44F2-99A3-0CC8C01D4DEE}" src="https://github.com/user-attachments/assets/3415c69f-2080-4b19-9b4d-afd0311f58e5" />

donde se utiliza directamente el postfix con mailx para poder adjuntar el HTML. Ya cuando el usuario abre el correo se ve así

<img width="606" height="211" alt="{6B445D0B-E9B8-4E23-9A4E-DAF6A3E83BB8}" src="https://github.com/user-attachments/assets/605d4b58-57b1-4edb-8529-6c8a32d55181" />

Este seria un ejemplo de como utlizar un servidor mail y las herramientas de parrot para simular un ataque 
