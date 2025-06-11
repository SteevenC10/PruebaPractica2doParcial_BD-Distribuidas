# 📘 Configuración Completa de Replica Set MongoDB con 3 VMs Ubuntu 22.04 LTS en GCP

Este documento detalla paso a paso cómo instalar y configurar un **Replica Set de MongoDB con lectura y escritura** entre tres máquinas virtuales en Google Cloud.

---

## 🖥️ Datos de las VMs (IP Internas)

| Nombre VM | IP Interna     |
|-----------|----------------|
| mongo1    | 10.128.0.2     |
| mongo2    | 10.128.0.3     |
| mongo3    | 10.128.0.4     |

---

## 🔐 Paso 1: Crear regla de firewall para MongoDB

Permitir tráfico en el puerto `27017` (requerido por MongoDB):

1. Ve a **VPC network > Firewall rules > Create firewall rule**
2. Configura lo siguiente:
   - **Name:** `allow-mongodb`
   - **Direction:** `Ingress`
   - **Targets:** `All instances in the network`
   - **Source IP ranges:** `10.128.0.0/20` (rango interno de tus VMs)
   - **Protocols and ports:** Marca `Specified protocols and ports` → TCP `27017`

---

## 🧰 Paso 2: Instalar MongoDB en cada VM

Ejecuta estos comandos en **cada una** de las VMs:

```bash
# Importar la clave pública
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Añadir el repositorio de MongoDB
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Instalar MongoDB
sudo apt update
sudo apt install -y mongodb-org

# Iniciar y habilitar el servicio
sudo systemctl enable mongod
sudo systemctl start mongod


# configurar mongod.conf
sudo nano /etc/mongod.conf

net:
  port: 27017
  bindIp: 0.0.0.0

replication:
  replSetName: rs0


#Inicializar el Replica Set (desde mongo1)
mongosh
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "10.128.0.2:27017" },
    { _id: 1, host: "10.128.0.3:27017" },
    { _id: 2, host: "10.128.0.4:27017" }
  ]
})


rs.status()


use testdb
db.ejemplo.insertOne({ mensaje: "Hola desde mongo1" })
