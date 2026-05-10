# **Guía de Despliegue: Intranet DNS \+ LDAP \+ FTP**

Este documento contiene los pasos técnicos detallados para reproducir la infraestructura de red del proyecto integrador, incluyendo la configuración de servicios críticos y las pruebas de verificación.

## **Tecnologías y Herramientas Usadas**

Para el desarrollo de este proyecto se han utilizado las siguientes tecnologías:

* **Sistema Operativo:** Ubuntu Server 22.04 LTS en todas las máquinas virtuales.  
* **Servicio DNS:** BIND9 (Berkeley Internet Name Domain).  
* **Servicio de Directorio:** OpenLDAP (slapd) para la gestión centralizada de usuarios.  
* **Servidor Web:** Apache2 con módulos de autenticación LDAP.  
* **Servidor FTP:** ProFTPD con soporte para módulos LDAP.  
* **Gestión de Red:** Netplan para la configuración de direccionamiento estático.  
* **Seguridad:** UFW (Uncomplicated Firewall) y permisos POSIX avanzados.  
* **Herramientas de Cliente:** FileZilla (FTP), Navegador Web (Intranet), nslookup y ldapsearch.

## **Configuración de Infraestructura (srv-infra \- 192.168.1.45)**

### **1\. DNS (BIND9)**

* **Instalación:** sudo apt install bind9  
* **Configuración de Zona:** Crear el archivo /etc/bind/db.centro.local definiendo los registros A para dns1, ldap, intranet y ftp.  
* **Configuración Global:** Editar /etc/bind/named.conf.options añadiendo:  
  * Forwarders: 8.8.8.8 y 8.8.4.4.  
  * dnssec-validation no;  
* **Seguridad:** sudo ufw allow bind9  
* **Reinicio:** sudo systemctl restart bind9

### **2\. LDAP (OpenLDAP)**

* **Instalación:** sudo apt install slapd ldap-utils  
* **Configuración:** sudo dpkg-reconfigure slapd estableciendo el dominio **centro.local**.  
* **Carga Inicial:** Importar la estructura mediante el archivo LDIF:  
  ldapadd \-x \-D "cn=admin,dc=centro,dc=local" \-W \-f carga\_inicial.ldif

* **Verificación:** sudo slapcat para confirmar que la base de datos es correcta.

## **Configuración de Servicios (srv-web \- 192.168.1.38)**

### **1\. Apache2 \+ Autenticación LDAP**

* **Instalación:** sudo apt install apache2  
* **Módulos:** Habilitar mediante sudo a2enmod ldap authnz\_ldap.  
* **VirtualHost:** Configurar /etc/apache2/sites-available/intranet.conf protegiendo las rutas /profesores y /alumnos con el grupo LDAP correspondiente.  
* **Activación:**  
  sudo a2ensite intranet.conf  
  sudo a2dissite 000-default.conf  
  sudo systemctl reload apache2

### **2\. FTP (ProFTPD)**

* **Instalación:** sudo apt install proftpd-basic proftpd-mod-ldap  
* **Gestión de Directorios:** Crear rutas en /srv/ftp/publicaciones/ y aplicar permisos POSIX:  
  sudo chown 11001:5001 /srv/ftp/publicaciones/profesores/profsaul  
  sudo chmod 750 /srv/ftp/publicaciones/profesores/profsaul

* **Integración LDAP:** \* En /etc/proftpd/modules.conf, habilitar mod\_ldap.c.  
  * Configurar /etc/proftpd/ldap.conf con el servidor ldap://192.168.1.45.  
* **Seguridad:** En proftpd.conf, activar DefaultRoot \~ y RequireValidShell off.  
* **Reinicio:** sudo systemctl restart proftpd

## **Pruebas de Verificación**

### **DNS**

Ejecutar desde el cliente para validar la resolución de nombres:

nslookup intranet.centro.local | nslookup ftp.centro.local

### **LDAP**

Comprobar la conectividad y búsqueda de usuarios:

ldapsearch \-x \-b "dc=centro,dc=local"

ldapwhoami \-H ldap://ldap.centro.local \-D "uid=profsaul,ou=usuarios,dc=centro,dc=local" \-x \-w 1234

### **Web e Intranet**

* Acceder a http://intranet.centro.local/profesores.  
* **Validación:** Debe aparecer el prompt de usuario/contraseña. Un alumno no debe poder entrar en la carpeta de profesores.

### **FTP y Almacenamiento**

* Conectar mediante **FileZilla** a ftp.centro.local.  
* Subir un archivo y verificar su existencia física en el servidor:  
  ls \-l /srv/ftp/publicaciones/profesores/profsaul/