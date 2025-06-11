
# PASOS PARA REALIZAR LA REPLICACION 

# Instalar Postgrest
1.- Instalamos postrest en ambas maquinas virtuales
--> sudo apt update
--> sudo apt install postgresql postgresql-contrib -y

2.- verificar la versión 
psql --version

3.- Editamos el archivo de configuración
--> sudo nano /etc/postgresql/14/main/postgresql.conf

    ingresamos lo siguiente: 
        listen_addresses = '*'
        wal_level = replica
        archive_mode = on
        archive_command = 'cp %p /var/lib/postgresql/14/archive/%f'
        max_wal_senders = 5
        wal_keep_size = 64
        synchronous_standby_names = 'standby1'
        synchronous_commit = on

4.- Editar el archivo de control de acceso en ambas VMs
--> sudo nano /etc/postgresql/14/main/pg_hba.conf

    4.1.- Agregamos lo siguiente: 
    host replication replicator 10.128.0.4/32 md5


5.-Reiniciamos el servicio.
--> sudo systemctl restart postgresql


6.- Crear el usuario replicador
--> sudo -u postgres psql
    Agregamos lo siguiente:
    
    CREATE ROLE replicator WITH REPLICATION LOGIN ENCRYPTED PASSWORD '1850622182';
\q

7.- Detener PostgreSQL y hacer backup en el SLAVE (VM2)
--> sudo systemctl stop postgresql
--> sudo rm -rf /var/lib/postgresql/14/main/*

Ahora realiza el backup desde VM1 (MASTER):

sudo -u postgres pg_basebackup -h 10.128.0.2 -D /var/lib/postgresql/14/main -U replicador -P --wal-method=stream



primary_conninfo = 'host=10.128.0.3 port=5432 user=replicador password=1850622182'
