---
title: Arquitectura de conectividad de una instancia administrada de Azure SQL Database | Microsoft Docs
description: Obtenga información sobre la comunicación de instancia administrada de Azure SQL Database y la arquitectura de conectividad también cómo los componentes de dirigir el tráfico a la instancia administrada.
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: fasttrack-edit
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, carlrab
manager: craigg
ms.date: 04/16/2019
ms.openlocfilehash: dbb5ee122e715aeaa66d786f02966beedd2447c3
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522335"
---
# <a name="connectivity-architecture-for-a-managed-instance-in-azure-sql-database"></a>Arquitectura de conectividad de una instancia administrada de Azure SQL Database

En este artículo se explica la comunicación en una instancia administrada de Azure SQL Database. También describe la arquitectura de conectividad y cómo los componentes de dirigir el tráfico a la instancia administrada.  

La instancia administrada de SQL Database se coloca dentro de la red virtual de Azure y la subred que está dedicada a las instancias administradas. Esta implementación proporciona:

- Una dirección IP privada segura.
- La capacidad de conectar una red local a una instancia administrada.
- La capacidad para conectarse a una instancia administrada en un servidor vinculado o en otro almacén de datos en local.
- La capacidad para conectarse a una instancia administrada a los recursos de Azure.

## <a name="communication-overview"></a>Información general sobre la comunicación

El siguiente diagrama muestra las entidades que se conectan a una instancia administrada. También muestra los recursos que necesitan para comunicarse con la instancia administrada. El proceso de comunicación en la parte inferior del diagrama representa las aplicaciones cliente y herramientas que se conectan a la instancia administrada como orígenes de datos.  

![Entidades de la arquitectura de conectividad](./media/managed-instance-connectivity-architecture/connectivityarch001.png)

Una instancia administrada es una plataforma como servicio (PaaS). Microsoft usa a automatizadas agents (administración, implementación y mantenimiento) para administrar este servicio basado en secuencias de datos de telemetría. Dado que Microsoft es responsable de administración, los clientes no pueden acceder a las máquinas virtuales del clúster de instancia administrada a través del protocolo de escritorio remoto (RDP).

Algunas operaciones a los usuarios finales o las aplicaciones pueden necesitar de SQL Server administra las instancias para interactuar con la plataforma. Un caso es la creación de una base de datos de instancia administrada. Este recurso se expone a través de la API de REST, PowerShell, CLI de Azure y Azure portal.

Las instancias administradas dependen de los servicios de Azure como Azure Storage para copias de seguridad, Azure Event Hubs para la telemetría, Azure Active Directory para la autenticación, Azure Key Vault para el cifrado de datos transparente (TDE) y un par de servicios de plataforma de Azure que proporcionan características de seguridad y compatibilidad. Las instancias administradas realiza las conexiones a estos servicios.

Todas las comunicaciones se cifran y firman utilizando certificados. Para comprobar la confiabilidad de comunicación partes, administra las instancias constantemente comprobación estos certificados a través de listas de revocación de certificados. Si se revocan los certificados, la instancia administrada cierra la conexión para proteger los datos.

## <a name="high-level-connectivity-architecture"></a>Arquitectura de conectividad de alto nivel

En un nivel alto, una instancia administrada es un conjunto de componentes del servicio. Estos componentes se hospedan en un conjunto dedicado de máquinas virtuales aisladas que se ejecutan dentro de la subred de red virtual del cliente. Estas máquinas forman un clúster virtual.

Un clúster virtual puede hospedar varias instancias administradas. Si es necesario, el clúster automáticamente expande o contrae cuando el cliente cambia el número de instancias aprovisionados en la subred.

Las aplicaciones cliente pueden conectarse a las instancias administradas y pueden consultar y actualizar las bases de datos dentro de la red virtual, red virtual emparejada, o redes conectadas mediante VPN o ExpressRoute de Azure. Esta red debe usar un punto de conexión y una dirección IP privada.  

![Diagrama de arquitectura de conectividad](./media/managed-instance-connectivity-architecture/connectivityarch002.png)

Servicios de implementación y administración de Microsoft que se ejecutan fuera de la red virtual. Una instancia administrada y servicios de Microsoft conectan a través de los puntos de conexión que tienen direcciones IP públicas. Cuando una instancia administrada crea una conexión de salida, en el extremo receptor realiza traducción de direcciones de red (NAT) el aspecto de conexión procede de esta dirección IP pública.

Tráfico de administración fluye a través de la red virtual del cliente. Esto significa que los elementos de la infraestructura de la red virtual pueden dañar el tráfico de administración mediante la realización de la instancia de un error y dejan de estar disponibles.

> [!IMPORTANT]
> Para mejorar la experiencia del cliente y la disponibilidad del servicio, Microsoft aplica una directiva de red intención en elementos de la infraestructura de red virtual de Azure. La directiva puede afectar al funcionamiento de la instancia administrada. Este mecanismo de plataforma transparente comunica los requisitos de red a los usuarios. Objetivo principal de la directiva es para evitar errores de configuración de red y para garantizar un funcionamiento normal de instancia administrada. Cuando se elimina una instancia administrada, también se quita la directiva de intención de la red.

## <a name="virtual-cluster-connectivity-architecture"></a>Arquitectura de conectividad del clúster virtual

Vamos a profundizar más en la arquitectura de conectividad para las instancias administradas. En el siguiente diagrama se muestra el diseño conceptual del clúster virtual.

![Arquitectura de conectividad del clúster virtual](./media/managed-instance-connectivity-architecture/connectivityarch003.png)

Los clientes se conectan a una instancia administrada mediante el uso de un nombre de host que tiene el formato `<mi_name>.<dns_zone>.database.windows.net`. Este nombre de host se resuelve en una dirección IP privada aunque se registra en una zona de sistema de nombres de dominio (DNS) de dominio público y es que pueda resolverse públicamente. El `zone-id` se genera automáticamente al crear el clúster. Si un clúster recién creado hospeda una instancia administrada secundaria, comparte su Id. de zona con el clúster principal. Para obtener más información, consulte [usar grupos de conmutación por error automática para permitir una recuperación de varias bases de datos transparente y coordinada](sql-database-auto-failover-group.md##enabling-geo-replication-between-managed-instances-and-their-vnets).

Esta dirección IP privada pertenece al equilibrador de carga interno de la instancia administrada. El equilibrador de carga dirige el tráfico a la puerta de enlace de la instancia administrada. Ya pueden ejecutar varias instancias administradas dentro del mismo clúster, la puerta de enlace usa el nombre de host de la instancia administrada para redirigir el tráfico al servicio de motor SQL correcto.

Servicios de implementación y administración de conexión a una instancia administrada mediante un [extremo de administración](#management-endpoint) que se asigna a una referencia externa del equilibrador de carga. Tráfico se enruta a los nodos solo si se hubiera recibido en un conjunto predefinido de puertos que utilizan sólo los componentes de administración de la instancia administrada. Un firewall integrado en los nodos está configurado para permitir el tráfico solo desde los intervalos IP de Microsoft. Todas las comunicaciones entre los componentes de administración y el plano de administración autentican mutuamente con certificados.

## <a name="management-endpoint"></a>Administración de puntos de conexión

Microsoft administra la instancia administrada mediante el uso de un extremo de administración. Este punto de conexión está dentro del clúster virtual de la instancia. El extremo de administración está protegido por un firewall integrado en el nivel de red. En el nivel de aplicación, está protegido mediante la comprobación del certificado mutuo. Para buscar la dirección IP del punto de conexión, consulte [determinar la dirección IP de la administración del punto de conexión](sql-database-managed-instance-find-management-endpoint-ip-address.md).

Al iniciar las conexiones dentro de la instancia administrada (como ocurre con las copias de seguridad y los registros de auditoría), el tráfico aparece iniciar desde la dirección IP pública de la administración del punto de conexión. Puede limitar el acceso a servicios públicos desde una instancia administrada mediante el establecimiento de reglas de firewall para permitir solo la dirección IP de la instancia administrada. Para obtener más información, consulte [Compruebe firewall integrado de la instancia administrada](sql-database-managed-instance-management-endpoint-verify-built-in-firewall.md).

> [!NOTE]
> Se optimiza el tráfico que se dirige a los servicios de Azure que están dentro de la región de la instancia administrada y por ese motivo no con NAT para administra dirección IP pública de punto de conexión de administración de instancia. Por ese motivo si necesita usar reglas de firewall basado en IP, con más frecuencia para el almacenamiento, servicios deben estar en una región distinta de la instancia administrada.

## <a name="network-requirements"></a>Requisitos de red

Implementar una instancia administrada en una subred dentro de la red virtual dedicada. La subred debe tener estas características:

- **Subred dedicada:** Subred de la instancia administrada no puede contener cualquier otro servicio en la nube que está asociado con él, y no puede ser una subred de puerta de enlace. La subred no puede contener cualquier recurso, pero la instancia administrada y versiones posteriores no se puede agregar otros tipos de recursos en la subred.
- **Grupo de seguridad de red (NSG):** Debe definir un NSG asociado a la red virtual [reglas de seguridad de entrada](#mandatory-inbound-security-rules) y [reglas de seguridad de salida](#mandatory-outbound-security-rules) antes de cualquier otra regla. Puede usar un NSG para controlar el acceso al punto de conexión de datos de la instancia administrada filtrando el tráfico en el puerto 1433 y puertos 11000 a 11999 cuando la instancia administrada está configurada para redirección las conexiones.
- **Tabla de usuario (UDR) de ruta definida por:** Una tabla UDR que está asociado con la red virtual debe incluir específico [entradas](#user-defined-routes).
- **Ningún punto de conexión de servicio:** Ningún extremo de servicio se debe asociar con la subred de la instancia administrada. Asegúrese de que la opción de puntos de conexión de servicio está deshabilitada cuando se crea la red virtual.
- **Suficientes direcciones IP:** La subred de instancia administrada debe tener al menos 16 direcciones IP. El mínimo recomendado es 32 direcciones IP. Para obtener más información, consulte [determinar el tamaño de la subred para instancias administradas](sql-database-managed-instance-determine-size-vnet-subnet.md). Puede implementar las instancias administradas en [la red existente](sql-database-managed-instance-configure-vnet-subnet.md) después de configurarlo para satisfacer [los requisitos de red para las instancias administradas](#network-requirements). De lo contrario, cree un [red y subred nuevas](sql-database-managed-instance-create-vnet-subnet.md).

> [!IMPORTANT]
> No se puede implementar una nueva instancia administrada si la subred de destino no tiene estas características. Cuando se crea una instancia administrada, se aplica una directiva de intención de red en la subred para evitar cambios no compatibles en el programa de instalación de red. Después de quita la última instancia de la subred, también se quita la directiva de intención de la red.

### <a name="mandatory-inbound-security-rules"></a>Reglas de seguridad de entrada obligatorias

| NOMBRE       |Port                        |Protocol|Origen           |Destino|.|
|------------|----------------------------|--------|-----------------|-----------|------|
|management  |9000, 9003, 1438, 1440, 1452|TCP     |Cualquiera              |MI SUBNET  |PERMITIR |
|mi_subnet   |Cualquiera                         |Cualquiera     |MI SUBNET        |MI SUBNET  |PERMITIR |
|health_probe|Cualquiera                         |Cualquiera     |AzureLoadBalancer|MI SUBNET  |PERMITIR |

### <a name="mandatory-outbound-security-rules"></a>Reglas de seguridad de salida obligatorias

| NOMBRE       |Port          |Protocol|Origen           |Destino|.|
|------------|--------------|--------|-----------------|-----------|------|
|management  |80, 443, 12000|TCP     |MI SUBNET        |AzureCloud |PERMITIR |
|mi_subnet   |Cualquiera           |Cualquiera     |MI SUBNET        |MI SUBNET  |PERMITIR |

> [!IMPORTANT]
> Asegúrese de que hay solo una regla de entrada para los puertos 9000, 9003, 1438, 1440, 1452 y una regla de salida de los puertos 80, 443, 12000. Administra el aprovisionamiento de la instancia a través de Azure Resource Manager implementaciones se producirá un error si las reglas entrantes y salientes se configuran por separado para cada puerto. Si estos puertos están siendo reglas independientes, se producirá un error en la implementación con código de error `VnetSubnetConflictWithIntendedPolicy`

\* MI subred hace referencia al intervalo de direcciones IP para la subred en el formulario de 10.x.x.x/y. Puede encontrar esta información en el portal de Azure, en Propiedades de la subred.

> [!IMPORTANT]
> Aunque las reglas de seguridad de entrada necesario permiten el tráfico desde _cualquier_ de código fuente en los puertos 9000, 9003, 1438, 1440 y 1452, estos puertos están protegidos por un firewall integrado. Para obtener más información, consulte [determinar la dirección del punto de conexión de administración](sql-database-managed-instance-find-management-endpoint-ip-address.md).
> [!NOTE]
> Si utiliza la replicación transaccional en una instancia administrada y usar cualquier instancia de la base de datos como un publicador o un distribuidor, abra el puerto 445 (salida TCP) en las reglas de seguridad de la subred. Este puerto permitirá el acceso al recurso compartido de archivos de Azure.

### <a name="user-defined-routes"></a>rutas definidas por el usuario

|NOMBRE|Prefijo de dirección|Próximo salto|
|----|--------------|-------|
|subnet_to_vnetlocal|MI SUBNET|Virtual network|
|mi-13-64-11-nexthop-internet|13.64.0.0/11|Internet|
|mi-13-96-13-nexthop-internet|13.96.0.0/13|Internet|
|mi-13-104-14-nexthop-internet|13.104.0.0/14|Internet|
|mi-20-8-nexthop-internet|20.0.0.0/8|Internet|
|mi-23-96-13-nexthop-internet|23.96.0.0/13|Internet|
|mi-40-64-10-nexthop-internet|40.64.0.0/10|Internet|
|mi-42-159-16-nexthop-internet|42.159.0.0/16|Internet|
|mi-51-8-nexthop-internet|51.0.0.0/8|Internet|
|mi-52-8-nexthop-internet|52.0.0.0/8|Internet|
|mi-64-4-18-nexthop-internet|64.4.0.0/18|Internet|
|mi-65-52-14-nexthop-internet|65.52.0.0/14|Internet|
|mi-66-119-144-20-nexthop-internet|66.119.144.0/20|Internet|
|mi-70-37-17-nexthop-internet|70.37.0.0/17|Internet|
|mi-70-37-128-18-nexthop-internet|70.37.128.0/18|Internet|
|mi-91-190-216-21-nexthop-internet|91.190.216.0/21|Internet|
|mi-94-245-64-18-nexthop-internet|94.245.64.0/18|Internet|
|mi-103-9-8-22-nexthop-internet|103.9.8.0/22|Internet|
|mi-103-25-156-22-nexthop-internet|103.25.156.0/22|Internet|
|mi-103-36-96-22-nexthop-internet|103.36.96.0/22|Internet|
|mi-103-255-140-22-nexthop-internet|103.255.140.0/22|Internet|
|mi-104-40-13-nexthop-internet|104.40.0.0/13|Internet|
|mi-104-146-15-nexthop-internet|104.146.0.0/15|Internet|
|mi-104-208-13-nexthop-internet|104.208.0.0/13|Internet|
|mi-111-221-16-20-nexthop-internet|111.221.16.0/20|Internet|
|mi-111-221-64-18-nexthop-internet|111.221.64.0/18|Internet|
|mi-129-75-16-nexthop-internet|129.75.0.0/16|Internet|
|mi-131-253-16-nexthop-internet|131.253.0.0/16|Internet|
|mi-132-245-16-nexthop-internet|132.245.0.0/16|Internet|
|mi-134-170-16-nexthop-internet|134.170.0.0/16|Internet|
|mi-134-177-16-nexthop-internet|134.177.0.0/16|Internet|
|mi-137-116-15-nexthop-internet|137.116.0.0/15|Internet|
|mi-137-135-16-nexthop-internet|137.135.0.0/16|Internet|
|mi-138-91-16-nexthop-internet|138.91.0.0/16|Internet|
|mi-138-196-16-nexthop-internet|138.196.0.0/16|Internet|
|mi-139-217-16-nexthop-internet|139.217.0.0/16|Internet|
|mi-139-219-16-nexthop-internet|139.219.0.0/16|Internet|
|mi-141-251-16-nexthop-internet|141.251.0.0/16|Internet|
|mi-146-147-16-nexthop-internet|146.147.0.0/16|Internet|
|mi-147-243-16-nexthop-internet|147.243.0.0/16|Internet|
|mi-150-171-16-nexthop-internet|150.171.0.0/16|Internet|
|mi-150-242-48-22-nexthop-internet|150.242.48.0/22|Internet|
|mi-157-54-15-nexthop-internet|157.54.0.0/15|Internet|
|mi-157-56-14-nexthop-internet|157.56.0.0/14|Internet|
|mi-157-60-16-nexthop-internet|157.60.0.0/16|Internet|
|mi-167-220-16-nexthop-internet|167.220.0.0/16|Internet|
|mi-168-61-16-nexthop-internet|168.61.0.0/16|Internet|
|mi-168-62-15-nexthop-internet|168.62.0.0/15|Internet|
|mi-191-232-13-nexthop-internet|191.232.0.0/13|Internet|
|mi-192-32-16-nexthop-internet|192.32.0.0/16|Internet|
|mi-192-48-225-24-nexthop-internet|192.48.225.0/24|Internet|
|mi-192-84-159-24-nexthop-internet|192.84.159.0/24|Internet|
|mi-192-84-160-23-nexthop-internet|192.84.160.0/23|Internet|
|mi-192-100-102-24-nexthop-internet|192.100.102.0/24|Internet|
|mi-192-100-103-24-nexthop-internet|192.100.103.0/24|Internet|
|mi-192-197-157-24-nexthop-internet|192.197.157.0/24|Internet|
|mi-193-149-64-19-nexthop-internet|193.149.64.0/19|Internet|
|mi-193-221-113-24-nexthop-internet|193.221.113.0/24|Internet|
|mi-194-69-96-19-nexthop-internet|194.69.96.0/19|Internet|
|mi-194-110-197-24-nexthop-internet|194.110.197.0/24|Internet|
|mi-198-105-232-22-nexthop-internet|198.105.232.0/22|Internet|
|mi-198-200-130-24-nexthop-internet|198.200.130.0/24|Internet|
|mi-198-206-164-24-nexthop-internet|198.206.164.0/24|Internet|
|mi-199-60-28-24-nexthop-internet|199.60.28.0/24|Internet|
|mi-199-74-210-24-nexthop-internet|199.74.210.0/24|Internet|
|mi-199-103-90-23-nexthop-internet|199.103.90.0/23|Internet|
|mi-199-103-122-24-nexthop-internet|199.103.122.0/24|Internet|
|mi-199-242-32-20-nexthop-internet|199.242.32.0/20|Internet|
|mi-199-242-48-21-nexthop-internet|199.242.48.0/21|Internet|
|mi-202-89-224-20-nexthop-internet|202.89.224.0/20|Internet|
|mi-204-13-120-21-nexthop-internet|204.13.120.0/21|Internet|
|mi-204-14-180-22-nexthop-internet|204.14.180.0/22|Internet|
|mi-204-79-135-24-nexthop-internet|204.79.135.0/24|Internet|
|mi-204-79-179-24-nexthop-internet|204.79.179.0/24|Internet|
|mi-204-79-181-24-nexthop-internet|204.79.181.0/24|Internet|
|mi-204-79-188-24-nexthop-internet|204.79.188.0/24|Internet|
|mi-204-79-195-24-nexthop-internet|204.79.195.0/24|Internet|
|mi-204-79-196-23-nexthop-internet|204.79.196.0/23|Internet|
|mi-204-79-252-24-nexthop-internet|204.79.252.0/24|Internet|
|mi-204-152-18-23-nexthop-internet|204.152.18.0/23|Internet|
|mi-204-152-140-23-nexthop-internet|204.152.140.0/23|Internet|
|mi-204-231-192-24-nexthop-internet|204.231.192.0/24|Internet|
|mi-204-231-194-23-nexthop-internet|204.231.194.0/23|Internet|
|mi-204-231-197-24-nexthop-internet|204.231.197.0/24|Internet|
|mi-204-231-198-23-nexthop-internet|204.231.198.0/23|Internet|
|mi-204-231-200-21-nexthop-internet|204.231.200.0/21|Internet|
|mi-204-231-208-20-nexthop-internet|204.231.208.0/20|Internet|
|mi-204-231-236-24-nexthop-internet|204.231.236.0/24|Internet|
|mi-205-174-224-20-nexthop-internet|205.174.224.0/20|Internet|
|mi-206-138-168-21-nexthop-internet|206.138.168.0/21|Internet|
|mi-206-191-224-19-nexthop-internet|206.191.224.0/19|Internet|
|mi-207-46-16-nexthop-internet|207.46.0.0/16|Internet|
|mi-207-68-128-18-nexthop-internet|207.68.128.0/18|Internet|
|mi-208-68-136-21-nexthop-internet|208.68.136.0/21|Internet|
|mi-208-76-44-22-nexthop-internet|208.76.44.0/22|Internet|
|mi-208-84-21-nexthop-internet|208.84.0.0/21|Internet|
|mi-209-240-192-19-nexthop-internet|209.240.192.0/19|Internet|
|mi-213-199-128-18-nexthop-internet|213.199.128.0/18|Internet|
|mi-216-32-180-22-nexthop-internet|216.32.180.0/22|Internet|
|mi-216-220-208-20-nexthop-internet|216.220.208.0/20|Internet|
||||

Además, puede agregar entradas a la tabla de enrutamiento para enrutar el tráfico que tiene intervalos de IP privadas de forma local como un destino a través de la puerta de enlace de red virtual o un dispositivo de red virtual (NVA).

Si la red virtual incluye un DNS personalizado, el servidor DNS personalizado debe ser capaz de resolver los nombres de host en \*. core.windows.net zona. Uso de características adicionales como autenticación de Azure AD puede requerir resolver FQDN adicionales. Para obtener más información, consulte [configurar un DNS personalizado](sql-database-managed-instance-custom-dns.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general, consulte [avanzadas de seguridad de los datos de SQL Database](sql-database-managed-instance.md).
- Obtenga información sobre cómo [configurar una nueva red virtual Azure](sql-database-managed-instance-create-vnet-subnet.md) o un [red virtual de Azure existente](sql-database-managed-instance-configure-vnet-subnet.md) donde puede implementar las instancias administradas.
- [Calcular el tamaño de la subred](sql-database-managed-instance-determine-size-vnet-subnet.md) donde desea implementar las instancias administradas.
- Obtenga información sobre cómo crear una instancia administrada:
  - Desde [Azure Portal](sql-database-managed-instance-get-started.md).
  - Mediante el uso de [PowerShell](scripts/sql-database-create-configure-managed-instance-powershell.md).
  - Mediante el uso de [una plantilla de Azure Resource Manager](https://azure.microsoft.com/resources/templates/101-sqlmi-new-vnet/).
  - Mediante el uso de [una plantilla de Azure Resource Manager (con el JumpBox, con SSMS incluidos)](https://portal.azure.com/). 
