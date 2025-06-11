 

Pasos Instalación de mysql en la maquina virtual Maestro / Esclavo 

# 1. Actualiza los repositorios 

--> sudo apt update 

# 2. Instala los paquetes necesarios para trabajar con repositorios HTTPS 

--> sudo apt install wget lsb-release gnupg -y 

# 3. Agrega el repositorio oficial de MySQL 

--> wget https://dev.mysql.com/get/mysql-apt-config_0.8.29-1_all.deb 

# 4. Instala el repositorio descargado 

--> sudo dpkg -i mysql-apt-config_0.8.29-1_all.deb 

 

# 5. Vuelve a actualizar los repositorios (ahora con los de MySQL) 

-->  sudo apt update  

# 6. Instala MySQL Server 

-->  sudo apt install mysql-server -y  

 

# Verifica que el servicio esté activo 

--> sudo systemctl status mysql 

 

# O en versiones más antiguas de Debian puedes usar: 

--> sudo service mysql status 

  
# Nos permite el ingreso al mysql dentro de la maquina virtual

sudo mysql -u root -p 

 

  
#Creación del usuario par apoder tener acceso al MYSQL de nuestra PC

CREATE USER 'admin'@'%' IDENTIFIED BY 'admin123'; 

CREATE DATABASE `centro-medico2` 
 
GRANT ALL PRIVILEGES ON `centro-medico2`.* TO 'admin'@'%'; 
 
# Reiniciamos nuestro sistema
sudo systemctl restart mysql 




PASOS PARA CREAR LA REPLICACION  ACTIVA Y PASIVA 

# Configurar el MAESTRO

1.- Preparación de la infraestructura

•Instancia Maestro: centro-medico2 con IP interna 10.128.0.5
•Instancia Esclavo: centro-exclavo con IP interna 10.128.0.6

2.- Editar el archivo de configuración de centros-médicos con el siguiente comando:
--> sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

    2.1- Dentro de este se añadira lo siguiente: 
           
            server-id = 1
            log_bin = /var/log/mysql/mysql-bin.log
            bind-address = 0.0.0.0
            binlog_do_db = replicationdb    

3.-Se reinicia Mysql con el siguiente comando:
--> sudo systemctl restart mysql

4.- Ingramos con el siguiente comando:
--> sudo mysql -u root -p

5.- Agregamos el usuario 

--> CREATE USER 'admin'@'%' IDENTIFIED WITH mysql_native_password BY '1850622182';
    GRANT REPLICATION SLAVE ON *.* TO 'admin'@'%';
    FLUSH PRIVILEGES;

6.- 




# CONFIGURAR EL ESCLAVO 

1.- Editar el archivo de configuración de centros-médicos con el siguiente comando:
--> sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

    1.1- Dentro de este se añadira lo siguiente: 
            server-id = 2
            relay-log = /var/log/mysql/mysql-relay-bin.log
            log_bin = /var/log/mysql/mysql-bin.log  
            read_only = 1

2.- Reiniciar el sistema
--> sudo systemctl restart mysql

3.-  Reconectamos al master 
--> sudo mysql -u root -p

    3.1.- Ingresamos lo siguiente: 

  CHANGE MASTER TO 
  MASTER_HOST='IP_MASTER',
  MASTER_USER='repl',
  MASTER_PASSWORD='replpass',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=1234;

START SLAVE;

4.- Verificar si esta ejecutando 
--> SHOW SLAVE STATUS\G

Confirmé que Slave_IO_Running: Yes 
Slave_SQL_Running: Yes


TEnemos la replica 

#En el maestro 
enviamos los mensajes de prueba;
1.-INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
2.-INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
3.- INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
4.- INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
5.- INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
6.-INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
7.-INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
8.-INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
9.-INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');
10.- INSERT INTO prueba_sync (mensaje) VALUES ('¡Hola desde el maestro');

Ingresmos un dato a la tabla de la base replicada 
# En el esclavo
revisamos el dato que se debio en viar en la base 