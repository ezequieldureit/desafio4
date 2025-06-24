# Desafío #4

**Fecha de entrega:** 23/06/2025

## Objetivo

El objetivo de este trabajo es poner en práctica todo lo aprendido sobre EC2, VPC y RDS.

## Escenario

Nuestra organización nos ha solicitado crear un nuevo entorno de desarrollo para un nuevo proyecto y debemos generar toda la documentación necesaria para que luego el equipo de implementación lo pueda crear en Staging y producción.

## Requisitos

Amazon RDS es el servicio que facilita la configuración, el funcionamiento y el escalado de bases de datos relacionales en AWS. Gracias a este servicio, el administrador de bases de datos puede delegar tareas operativas como la gestión del sistema operativo, el almacenamiento, copias de seguridad, alta disponibilidad, entre otras.

Antes de crear la instancia de la BD, se nos pide realizar una serie de acciones:

1. Crear un usuario IAM con permisos de administrador. Será con este usuario con el que realicemos el resto del tutorial. Amazon aconseja no usar nunca el usuario raíz, salvo ocasiones puntuales, y guardar en lugar seguro sus credenciales.

   * Pueden usar AWS Academy, en ese caso no es necesario crear un usuario.

2. La instancia de BD se creará en una VPC (Virtual Private Cloud). Por lo tanto, también es necesario definir las reglas de grupo de seguridad para tener acceso a esta VPC (que seguramente será del tipo EC2-VPC).

3. Elegiremos el escenario con la instancia de BD en una VPC y una aplicación cliente a través de Internet.

## Crear la VPC

Ahora deberá crear una VPC para utilizarla con una instancia de base de datos, con la configuración descrita en el punto anterior:

* Abra la consola de Amazon VPC en [https://console.aws.amazon.com/vpc/](https://console.aws.amazon.com/vpc/).
* En la esquina superior derecha de la Consola de administración de AWS, elija la región en la que desea crear la VPC.
* Cree manualmente la VPC con los siguientes datos:

  * IPv4 CIDR block: 10.0.0.0/16
  * IPv6 CIDR block: No IPv6 CIDR Block
  * VPC name: tutorial-vpc
* Cree una Subnet pública:

  * Public subnet’s IPv4 CIDR: 10.0.0.0/24
  * Availability Zone: No preference
  * Public subnet name: Tutorial public

Al crear estos recursos, también se deben generar:

* Route Table
* Internet Gateway
* Network ACL
* Security Group

## Configurar el Security Group

* Vaya a "Security Groups" y seleccione el grupo asociado a la VPC.
* En "Inbound rules", edite las reglas:

  * Tipo: MYSQL/Aurora
  * Protocolo: TCP
  * Puerto: 3306
  * Fuente: 0.0.0.0/0 (o su IP para mayor seguridad)

Ejemplo seguro:

* Tipo: MYSQL/Aurora
* Protocolo: TCP
* Puerto: 3306
* Fuente: 56.176.2.108/32

## Crear subred adicional

Se requieren dos subredes para crear un DB Subnet Group. Como esta base será pública, agregaremos una segunda subred pública:

* VPC: tutorial-vpc
* Name tag: Tutorial public 2
* Availability Zone: us-west-2b
* IPv4 CIDR block: 10.0.2.0/24
* Asociar la misma Route Table de la primera subred

## Crear grupo de subredes de base de datos

* Consola: Amazon RDS > Subnet groups > Create DB Subnet Group
* Name: tutorial-db-subnet-group
* Description: Tutorial DB Subnet Group
* VPC: tutorial-vpc
* Subnets: Tutorial public y Tutorial public 2 (en us-west-2a y us-west-2b)

## Crear la instancia de base de datos en la VPC

1. Ir a Amazon RDS > Databases > Create database
2. Modo: Standard Create
3. Engine: MariaDB
4. DB instance size: Free Tier
5. DB identifier: tutorial-db
6. Usuario admin y contraseña (autogenerada o manual)
7. VPC: tutorial-vpc
8. Subnet Group: tutorial-db-subnet-group
9. Public access: Yes
10. Asociar Security Group configurado previamente
11. Crear base de datos
12. Ver credenciales y copiar endpoint generado

## Verificar acceso a la instancia

Usar un cliente MariaDB o consola:

```bash
mariadb -h mariadbinstancia.skdimeitllwst.us-west-1.rds.amazonaws.com -u username -p
```

## Posibles problemas

* Revisar que la VPC tenga activado:

  * DNS resolution
  * DNS hostnames
* Asegurarse de que la Route Table tenga ambas subredes asociadas
* Verificar reglas del Security Group

## Mejorar la seguridad

En lugar de permitir acceso público, se puede:

* Poner la base en una subnet privada
* Usar un bastion host SSH dentro de la VPC

## Entregables

1. Documento con los pasos realizados (puede incluir CLI)
2. Diagrama de arquitectura detallado

## Evaluación

* Entrega en fecha
* Documentación clara y comprensible
* Material de soporte (diagrama)
* Cumplimiento de consignas
* Entregable funcional (ej. script ejecutable sin errores)
