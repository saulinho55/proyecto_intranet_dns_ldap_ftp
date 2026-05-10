-Guía de Despliegue: Intranet DNS + LDAP + FTP-
Se detalla los pasos técnicos para reproducir la infraestructura de red del proyecto.

DNS (BIND9)
    Instalación: sudo apt install bind9
    Crear zona:  Crear /etc/bind/db.centro.local con registros A para dns1, ldap, intranet y ftp apuntando a las IPs correspondientes de las MVs.
    Activación: Declarar la zona en /etc/bind/named.conf.default-zones
    Firewall: sudo ufw allow bind9 para abrir los puertos
    Forwarders: En /etc/bind/named.conf.options, añadir los DNS (8.8.8.8(8.8.4.4)) y dnssec-validation para asegurar la respuesta.
    Reinicio: sudo systemctl restart bind9
    Revisar estado: sudo systemctl status bind9

LDAP
    Instalación: sudo apt install slapd ldap-utils
    Verificación: sudo slapcat para comprobar que se ha creado correctamente
    Configuración: sudo dpkg-reconfigure slapd (Dominio: centro.local)
    Crear archivo .ldif: sudo nano carga_inicial.ldif
    Cargar Datos: ldapadd -x -D "cn=admin,dc=centro,dc=local" -W -f carga_inicial.ldif

Apache + Autenticación LDAP
    Instalación: sudo apt install apache2
    VirtualHost: Configurar /etc/apache2/sites-available/intranet.conf
    Habilitar Módulos: sudo a2enmod ldap authnz_ldap
    Activación del sitio: sudo a2ensite intranet.conf / sudo systemctl reload apache2

FTP (ProFTPD)
    Instalación: sudo apt install proftpd-basic proftpd-mod-ldap
    Directorios: Crear rutas de los usuarios en /srv/ftp/publicaciones/ y asignar los IDs del individuo y grupo de LDAP
    Módulos: sudo nano /etc/proftpd/modules.conf, descomentar el módulo mod_ldap.c
    Configuración LDAP: Editar /etc/proftpd/ldap.conf con los parámetros de conexión al servidor LDAP
    Editar proftpd.conf: Descomentar DefaultRoot ~ y RequireValidShell off
    Reiniciar: sudo systemctl restart proftpd


-Pruebas de Verificación-

DNS
    ping/nslookup intranet.centro.local/ftp.centro.local/dns1.centro.local/ldap.centro.local

LDAP
    ldapsearch -x -b "dc=centro,dc=local" (Verifica estructura completa)
    ldapwhoami -H ldap://ldap.centro.local -D "uid=profsaul,ou=usuarios,dc=centro,dc=local" -w 1234 (Devuelve el DN del usuario)

Web y FTP
    Acceder a http://intranet.centro.local/(profesores/alumnos) - Verificar si sale para meter usuario y contraseña. Verificar que entra en el index.
    Conexión con FileZilla a ftp.centro.local (usuario y contraseña) - Prueba a enviar archivo y buscar con ls /srv/ftp/publicaciones/ si se envio correctamente a la carpeta especifica