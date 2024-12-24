# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:latest

# ------------------------- 1. CONFIGURACIONES GENERALES E INSTALACIONES -------------------------
# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host" \
    maintainer="usuarioBIRT <usuarioBIRT@birt.eus>"

# Actualizamos la lista de paquetes e instalamos nano y apache2
# Actualizamos la lista de paquetes
RUN apt-get update && apt-get upgrade -y

# Instalamos paquetes necesarios
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 nano proftpd openssh-server openssl git

#Instala mod-crypto, que contiene mod_tls
RUN apt-get install proftpd-mod-crypto -y

# Limpiamos los archivos temporales de apt
RUN rm -rf /var/lib/apt/lists/*

# ------------------------- 2. CONFIGURACIONES DE APACHE -------------------------

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitio1
RUN mkdir -p /var/www/html/sitio2

# Copiamos archivos de configuración al contenedor
COPY apache/web/indexPrimero.html apache/web/indexSegundo.html \
     apache/conf/sitioPrimero.conf apache/conf/sitioSegundo.conf \
     apache/conf/certs/sitioPrimero.key apache/conf/certs/sitioPrimero.cer \
     /

# Movemos los archivos a sus ubicaciones adecuadas
RUN mv /indexPrimero.html /var/www/html/sitio1/index.html && \
    mv /indexSegundo.html /var/www/html/sitio2/index.html && \
    mv /sitioPrimero.conf /etc/apache2/sites-available/sitioPrimero.conf && \
    mv /sitioSegundo.conf /etc/apache2/sites-available/sitioSegundo.conf && \
    mv /sitioPrimero.key /etc/ssl/private/sitioPrimero.key && \
    mv /sitioPrimero.cer /etc/ssl/certs/sitioPrimero.cer

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioPrimero.conf
RUN a2ensite sitioSegundo.conf
RUN a2enmod ssl

# Exponemos los puertos 80 y 43
EXPOSE 80
EXPOSE 443

# ------------------------- 3. CONFIGURACIONES DEL FTP -------------------------

# Creamos directorio para la configuración del TLS
RUN mkdir /etc/proftpd/tls

# Crea el usuario nargello1 y su contraseña, que será también nargello1
  # Crea el usuario y le asigna /var/www/html/sitio1 como carpeta de trabajo
  RUN useradd -d /var/www/html/sitio1 -s /bin/bash nargello1
  # Establece la contraseña nargello1 para el usuario nargello1
  RUN echo "nargello1:nargello1" | chpasswd
  #Especifica que el usuario y el grupo propietario de /var/www/html/sitio1 sea nargello1
  RUN chown nargello1:nargello1 /var/www/html/sitio1

# Sustituye los ficheros proftpd.conf y tls.conf previamente configurados de la carpeta conf al contenedor
COPY ftp/conf/proftpd.conf /etc/proftpd/proftpd.conf
COPY ftp/conf/tls.conf /etc/proftpd/tls.conf

#Descomenta LoadModule mod_tls.c en modules.conf
RUN sed -i 's/#LoadModule mod_tls.c/LoadModule mod_tls.c/' /etc/proftpd/modules.conf

# Copia el certificado autofirmado y la llave al contenedor
COPY ftp/conf/certs/cert.pem /etc/proftpd/tls
COPY ftp/conf/certs/key.pem /etc/proftpd/tls

# Expone los puertos 21 y 50000-50030
EXPOSE 21 50000-50030

# ------------------------- 3. CONFIGURACIONES DEL SSH -------------------------

# Crea el usuario nargello2 y su contraseña, que será también nargello2
    # Crea el usuario y le asigna /var/www/html/sitio2 como carpeta de trabajo
    RUN useradd -d /var/www/html/sitio2 -s /bin/bash nargello2
    # Establece la contraseña nargello1 para el usuario nargello1
    RUN echo "nargello2:nargello2" | chpasswd
    #Especifica que el usuario y el grupo propietario de /var/www/html/sitio1 sea nargello1
    RUN chown nargello2:nargello2 /var/www/html/sitio2

# Sustituye el fichero sshd_config previamente configurado de la carpeta conf al contenedor
COPY ssh/conf/sshd_config /etc/ssh/sshd_config

# Expone el puerto 2222
EXPOSE 2222

#------------------------- 3. CONFIGURACIONES DE GIT -------------------------
# Crear el directorio repo_git
RUN mkdir -p /var/ftp/pub/repo_git
# Hacer a los usuarios anónimos propietarios del directorio
RUN chown ftp:nogroup /var/ftp/pub/repo_git
#Conceder permisos para que puedan acceder
RUN chmod 755 /var/ftp/pub/repo_git

COPY git/id_rsa /etc

RUN apt-get install openssh-client --yes

#RUN eval "$(ssh-agent -s)" \
#&& chmod 600 /etc/id_rsa \
#&& ssh-add /etc/id_rsa \
#&& ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
#&& git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /var/ftp/pub/repo_git

#  ------------------------- 4. INICIO DEL CONTENEDOR -------------------------  

# Comando por defecto al iniciar el contenedor
CMD ["apache2-foreground"]