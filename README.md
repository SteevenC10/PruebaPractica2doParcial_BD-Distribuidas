# ☁️ Distributed Databases Replication Lab ☁️

Este repositorio describe los pasos para configurar una Virtual Private Cloud (VPC) en Google Cloud Platform (GCP) con una subred privada y una regla de firewall que permita la comunicación entre máquinas virtuales para la replicación distribuida de bases de datos.

---

## 🚀 Getting Started

Esta guía te llevará a través de la creación de una red VPC personalizada y la configuración de una regla de firewall para permitir el tráfico esencial de bases de datos.

---

## 🛠️ Setup Instructions

### 1. Crear la Red VPC

Primero, crearemos una red VPC personalizada para alojar nuestro entorno de replicación.

- Navega a GCP: Ve a **VPC Network > Create VPC Network**.
- Configura los detalles de la red:
  - **Name:** replication-lab-vpc
  - **Subnet Creation Mode:** Custom
  - **Subnet Name:** subnet-replication
  - **Region:** Elige una región (ejemplo: us-central1)
  - **IP Range:** 10.128.0.0/20
- Haz clic en **Create** para crear la red VPC.

---

### 2. Configurar Regla de Firewall

Configura una regla de firewall para permitir los puertos específicos de las bases de datos (MySQL, SQL Server, MongoDB) para que puedan comunicarse dentro de la VPC.

- Navega a GCP: Ve a **VPC Network > Firewall**.
- Haz clic en **Create Firewall Rule**.
- Configura la regla:
  - **Name:** allow-db-ports
  - **Network:** replication-lab-vpc
  - **Targets:** All instances in the network
  - **Source IP Ranges:** 10.128.0.0/20
  - **Protocols and ports:**  
    Selecciona *Specified protocols and ports*, marca **tcp** e ingresa los puertos:  
    `3306, 1433, 27017`
- Haz clic en **Create** para guardar la regla.

---

### 3. Crear Instancias de VM

- Crea varias instancias de VM en la subred `subnet-replication` para simular nodos de base de datos distribuidos.
- Usa imágenes con las bases de datos que quieres probar (MySQL, SQL Server, MongoDB) o instala manualmente en VMs Linux.
- Asigna solo IPs internas para mantener el tráfico dentro de la VPC.

---

### 4. Configurar DNS Interno / Hostnames

- Utiliza el DNS interno de GCP o configura hostnames estáticos para que los nodos se resuelvan entre sí fácilmente.  
- Ejemplo: `mysql-node1.replication-lab.internal`

---

### 5. Configurar Bases de Datos

- Configura cada base de datos para aceptar conexiones solo desde IPs internas (bind a 0.0.0.0 o IP interna).  
- Crea usuarios específicos para replicación con permisos adecuados (ejemplo: en MySQL, usuario con permiso `REPLICATION SLAVE`).

---

### 6. Monitoreo y Logs

- Habilita Stackdriver (Cloud Monitoring y Logging) para monitorear el tráfico y logs de base de datos.  
- Útil para resolver problemas de replicación o conectividad.

---

## 🔄 Opcionales Avanzados

- **Private Google Access:** Para que las VMs sin IP pública accedan a APIs de Google.  
- **Cloud NAT:** Si las VMs requieren internet para actualizaciones pero sin IP pública.  
- **Subredes Multi-región:** Para probar replicación geográfica creando subredes en varias regiones.

---

## Ejemplo usando gcloud CLI

```bash
# Crear la red VPC
gcloud compute networks create replication-lab-vpc --subnet-mode=custom

# Crear la subred
gcloud compute networks subnets create subnet-replication \
  --network=replication-lab-vpc \
  --region=us-central1 \
  --range=10.128.0.0/20

# Crear regla de firewall para puertos de base de datos
gcloud compute firewall-rules create allow-db-ports \
  --network=replication-lab-vpc \
  --allow tcp:3306,tcp:1433,tcp:27017 \
  --source-ranges=10.128.0.0/20 \
  --description="Allow MySQL, SQL Server, MongoDB traffic"
