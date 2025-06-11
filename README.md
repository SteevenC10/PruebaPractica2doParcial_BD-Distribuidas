# PruebaPractica2doParcial_BD-Distribuidas

#Creacion de la red VPC Teniendo encuenta las siguientes caracteristicas: 
 * VPC con subred privada (10.128.0.0/20) y regla de firewall que permita TCP/3306, 
1433, 27017 entre las VMs

# PASOS PARA AGREGAR LA RED CPC 
  1.  Ve a GCP > VPC Network > "Create VPC Network"

Nombre: replication-lab-vpc

Subred personalizada:

Nombre: subnet-replication

Región: selecciona una (ej. us-central1)

Rango IP: 10.128.0.0/20

Crea la VPC.
