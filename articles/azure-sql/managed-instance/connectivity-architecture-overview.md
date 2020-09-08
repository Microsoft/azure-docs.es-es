---
title: Arquitectura de conectividad
titleSuffix: Azure SQL Managed Instance
description: Obtenga información sobre la arquitectura de comunicación y conectividad de Azure SQL Managed Instance y sobre la forma en que los componentes dirigen el tráfico a una instancia administrada.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: fasttrack-edit
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, carlrab
ms.date: 03/17/2020
ms.openlocfilehash: 059828336288eeadc0567fed060db07e323f885c
ms.sourcegitcommit: f1b18ade73082f12fa8f62f913255a7d3a7e42d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88761872"
---
# <a name="connectivity-architecture-for-azure-sql-managed-instance"></a>Arquitectura de conectividad de Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se explica la comunicación en Azure SQL Managed Instance. También se describe la arquitectura de conectividad y cómo los componentes dirigen el tráfico a una instancia administrada.  

SQL Managed Instance se coloca dentro de la red virtual de Azure y de la subred dedicada a las instancias administradas. Esta implementación proporciona:

- Una dirección IP privada segura.
- La posibilidad de conectar una red local a SQL Managed Instance.
- La posibilidad de conectar SQL Managed Instance a un servidor vinculado o a otro almacén de datos local.
- La posibilidad de conectar SQL Managed Instance a recursos de Azure.

## <a name="communication-overview"></a>Información general sobre la comunicación

En el siguiente diagrama se muestran las entidades que se conectan a SQL Managed Instance. También se muestran los recursos que deben comunicarse con una instancia administrada. El proceso de comunicación de la parte inferior del diagrama representa las aplicaciones y las herramientas del cliente que se conectan a SQL Managed Instance como orígenes de datos.  

![Entidades de la arquitectura de conectividad](./media/connectivity-architecture-overview/connectivityarch001.png)

SQL Managed Instance es una oferta de plataforma como servicio (PaaS). Azure emplea agentes automatizados (administración, implementación y mantenimiento) para administrar este servicio en función de flujos de datos de telemetría. Dado que Azure es responsable de la administración, los clientes no pueden acceder a las máquinas del clúster virtual de Instancia administrada de SQL mediante el Protocolo de escritorio remoto (RDP).

Algunas operaciones iniciadas por los usuarios finales o las aplicaciones podrían requerir que SQL Managed Instance interactuara con la plataforma. Un caso es la creación de una base de datos de Instancia administrada de SQL. Este recurso se expone mediante Azure Portal, PowerShell, la CLI de Azure y la API REST.

SQL Managed Instance depende de servicios de Azure como Azure Storage para las copias de seguridad, Azure Event Hubs para la telemetría, Azure Active Directory (Azure AD) para la autenticación, Azure Key Vault para el Cifrado de datos transparente (TDE) y un par de servicios de plataforma de Azure que proporcionan características de seguridad y compatibilidad. SQL Managed Instance crea conexiones a estos servicios.

Todas las comunicaciones se cifran y firman mediante certificados. Para comprobar la confiabilidad de las partes en comunicación, SQL Managed Instance comprueba constantemente estos certificados por medio de listas de revocación de certificados. Si los certificados se revocan, SQL Managed Instance cierra la conexión para proteger los datos.

## <a name="high-level-connectivity-architecture"></a>Arquitectura de conectividad de alto nivel

En general, SQL Managed Instance es un conjunto de componentes del servicio. Estos componentes se hospedan en un conjunto dedicado de máquinas virtuales aisladas que se ejecutan dentro de la subred de red virtual del cliente. Estas máquinas forman un clúster virtual.

Un clúster virtual puede hospedar varias instancias administradas. El clúster se expande y contrae automáticamente si es necesario cuando el cliente cambia el número de instancias aprovisionadas en la subred.

Las aplicaciones de los clientes pueden conectarse a SQL Managed Instance y pueden consultar y actualizar las bases de datos dentro de la red virtual, la red virtual emparejada o la red conectada mediante VPN o Azure ExpressRoute. Esta red debe usar un punto de conexión y una dirección IP privada.  

![Diagrama de la arquitectura de conectividad](./media/connectivity-architecture-overview/connectivityarch002.png)

Los servicios de implementación y administración de Azure se ejecutan fuera de la red virtual. SQL Managed Instance y los servicios de Azure se conectan a través de los puntos de conexión que tienen direcciones IP públicas. Cuando SQL Managed Instance crea una conexión de salida, en el extremo receptor parece que procede de esta dirección IP pública debido a la traducción de direcciones de red (NAT).

El tráfico de administración fluye a través de la red virtual del cliente. Esto significa que los elementos de la infraestructura de la red virtual pueden dañar el tráfico de administración al provocar que la instancia genere un error y deje de estar disponible.

> [!IMPORTANT]
> Para mejorar la experiencia del cliente y la disponibilidad del servicio, Azure aplica una directiva de intención de red a los elementos de la infraestructura de la red virtual de Azure. La directiva puede afectar al modo en que funciona SQL Managed Instance. Este mecanismo de plataforma comunica de forma transparente los requisitos de red a los usuarios. El objetivo principal de la directiva es evitar errores de configuración de red y garantizar el funcionamiento normal de Instancia administrada de SQL. Cuando se elimina una instancia administrada, también se quita la directiva de intención de red.

## <a name="virtual-cluster-connectivity-architecture"></a>Arquitectura de conectividad del clúster virtual

Vamos a profundizar más en la arquitectura de conectividad de SQL Managed Instance. En el siguiente diagrama se muestra el diseño conceptual del clúster virtual.

![Arquitectura de conectividad del clúster virtual](./media/connectivity-architecture-overview/connectivityarch003.png)

Los clientes se conectan a SQL Managed Instance mediante un nombre de host que tiene el formato `<mi_name>.<dns_zone>.database.windows.net`. Este nombre de host se resuelve en una dirección IP privada, aunque se registra en la zona DNS (Sistema de nombres de dominio) pública y puede resolverse públicamente. `zone-id` se genera automáticamente al crear el clúster. Si un clúster recién creado hospeda una instancia administrada secundaria, comparte su identificador de zona con el clúster principal. Para más información, consulte [Uso de grupos de conmutación por error automática para permitir la conmutación por error de varias bases de datos de manera transparente y coordinada](../database/auto-failover-group-overview.md#enabling-geo-replication-between-managed-instances-and-their-vnets).

Esta dirección IP privada pertenece al equilibrador de carga interno de SQL Managed Instance. El equilibrador de carga dirige el tráfico a la puerta de enlace de SQL Managed Instance. Como se pueden ejecutar varias instancias administradas dentro del mismo clúster, la puerta de enlace usa el nombre de host de SQL Managed Instance para dirigir el tráfico al servicio correcto del motor de SQL.

Los servicios de administración e implementación se conectan a SQL Managed Instance mediante un [punto de conexión de administración](#management-endpoint) que se asigna a un equilibrador de carga externo. El tráfico se enruta a los nodos únicamente si se recibe en un conjunto predefinido de puertos que solo usan los componentes de administración de SQL Managed Instance. Un firewall integrado en los nodos se configura para permitir el tráfico solo desde intervalos IP de Microsoft. Los certificados autentican mutuamente toda la comunicación entre los componentes de administración y el plano de administración.

## <a name="management-endpoint"></a>Administración de puntos de conexión

Azure administra SQL Managed Instance mediante un punto de conexión de administración. Este punto de conexión está dentro del clúster virtual de una instancia. El punto de conexión de administración está protegido por un firewall integrado en el nivel de red. En el nivel de aplicación, está protegido por la comprobación mutua del certificado. Para buscar la dirección IP del punto de conexión, consulte [Determinación de la dirección IP del punto de conexión de administración](management-endpoint-find-ip-address.md).

Cuando las conexiones se inician dentro de SQL Managed Instance (como en las copias de seguridad y los registros de auditoría), el tráfico parece iniciarse desde la dirección IP pública del punto de conexión de administración. Se puede limitar el acceso a los servicios públicos de SQL Managed Instance mediante el establecimiento de reglas de firewall que permitan únicamente la dirección IP de SQL Managed Instance. Para obtener más información, consulte [Comprobación del firewall integrado de SQL Managed Instance](management-endpoint-verify-built-in-firewall.md).

> [!NOTE]
> El tráfico que se dirige a servicios de Azure que están dentro de la región de SQL Managed Instance se optimiza y, por ese motivo, no se dirige mediante NAT a la dirección IP pública del punto de conexión de administración. Así que, si necesita usar reglas de firewall basadas en IP, normalmente para el almacenamiento, el servicio debe estar en una región distinta de la de SQL Managed Instance.

## <a name="service-aided-subnet-configuration"></a>Configuración de subred asistida por servicio

Para cumplir los requisitos de manejabilidad y seguridad de los clientes, SQL Managed Instance cambia de una configuración de subred manual a una asistida por servicio.

Con la configuración de subred asistida por servicio, el usuario tiene control total sobre el tráfico de datos (TDS) a la vez que SQL Managed Instance asume la responsabilidad de garantizar el flujo ininterrumpido de tráfico de administración para cumplir el Acuerdo de Nivel de Servicio.

La configuración de la subred asistida por servicio se basa en la característica de [delegación de subred](../../virtual-network/subnet-delegation-overview.md) de la red virtual para proporcionar administración automática de la configuración de red y habilitar puntos de conexión de servicio. 

Los puntos de conexión de servicio podrían usarse para configurar reglas de firewall de red virtual en cuentas de almacenamiento que conservan copias de seguridad y registros de auditoría. Incluso si tienen habilitados los puntos de conexión del servicio, se recomienda que los clientes utilicen el [vínculo privado](../../private-link/private-link-overview.md) que añade más seguridad a los puntos de conexión del servicio.

> [!IMPORTANT]
> Debido a las características de configuración del plano de control, la configuración de subred asistida por servicio no habilitaría los puntos de conexión del servicio en nubes nacionales. 

### <a name="network-requirements"></a>Requisitos de red

Implemente SQL Managed Instance en una subred dedicada dentro de la red virtual. La subred debe tener estas características:

- **Subred dedicada:** la subred de SQL Managed Instance no puede contener ningún otro servicio en la nube asociado a ella ni puede ser una subred de puerta de enlace. La subred no puede contener ningún recurso, a excepción de SQL Managed Instance, ni se le pueden agregar posteriormente otros tipos de recursos.
- **Delegación de subred:** la subred de SQL Managed Instance debe delegarse en el proveedor de recursos `Microsoft.Sql/managedInstances`.
- **Grupo de seguridad de red (NSG):** se debe asociar un grupo de seguridad de red a la subred de SQL Managed Instance. Puede usar un NSG para controlar el acceso al punto de conexión de datos de SQL Managed Instance mediante el filtrado del tráfico en el puerto 1433 y en los puertos 11000 a 11999 cuando SQL Managed Instance se configura para conexiones de redirección. El servicio aprovisionará y mantendrá automáticamente las [reglas](#mandatory-inbound-security-rules-with-service-aided-subnet-configuration) actuales necesarias para permitir el flujo ininterrumpido de tráfico de administración.
- **Tabla de rutas definida por el usuario (UDR):** se debe asociar una tabla UDR a la subred de SQL Managed Instance. Puede agregar entradas a la tabla de rutas para enrutar el tráfico que tiene intervalos IP privados locales como destino a través de una puerta de enlace de red virtual o de un dispositivo de red virtual (NVA). El servicio aprovisionará y mantendrá automáticamente las [entradas](#user-defined-routes-with-service-aided-subnet-configuration) actuales necesarias para permitir el flujo ininterrumpido de tráfico de administración.
- **Suficientes direcciones IP:** la subred de Instancia administrada de SQL debe tener al menos 16 direcciones IP. El mínimo recomendado es 32 direcciones IP. Para obtener más información, consulte [Determinación del tamaño de la subred de SQL Managed Instance](vnet-subnet-determine-size.md). Puede implementar instancias administradas en [la red existente](vnet-existing-add-subnet.md) después de configurarla para satisfacer [los requisitos de red de SQL Managed Instance](#network-requirements). De lo contrario, cree [una red y una subred](virtual-network-subnet-create-arm-template.md).

> [!IMPORTANT]
> Cuando se crea una instancia administrada, se aplica una directiva de intención de red en la subred para evitar cambios no compatibles con la configuración de red. Después de quitar la última instancia de la subred, también se quitará la directiva de intención de red.

### <a name="mandatory-inbound-security-rules-with-service-aided-subnet-configuration"></a>Reglas de seguridad de entrada obligatorias con la configuración de subred asistida por servicio

| Nombre       |Port                        |Protocolo|Source           |Destination|Acción|
|------------|----------------------------|--------|-----------------|-----------|------|
|management  |9000, 9003, 1438, 1440, 1452|TCP     |SqlManagement    |MI SUBNET  |Allow |
|            |9000, 9003                  |TCP     |CorpnetSaw       |MI SUBNET  |Allow |
|            |9000, 9003                  |TCP     |CorpnetPublic    |MI SUBNET  |Allow |
|mi_subnet   |Any                         |Any     |MI SUBNET        |MI SUBNET  |Allow |
|health_probe|Any                         |Any     |AzureLoadBalancer|MI SUBNET  |Allow |

### <a name="mandatory-outbound-security-rules-with-service-aided-subnet-configuration"></a>Reglas de seguridad de entrada obligatorias con la configuración de subred asistida por servicio

| Nombre       |Port          |Protocolo|Source           |Destination|Acción|
|------------|--------------|--------|-----------------|-----------|------|
|management  |443, 12000    |TCP     |MI SUBNET        |AzureCloud |Allow |
|mi_subnet   |Any           |Any     |MI SUBNET        |MI SUBNET  |Allow |

### <a name="user-defined-routes-with-service-aided-subnet-configuration"></a>Rutas definidas por el usuario con la configuración de subred asistida por servicio

|Nombre|Prefijo de dirección|Próximo salto|
|----|--------------|-------|
|subnet-to-vnetlocal|MI SUBNET|Virtual network|
|mi-13-64-11-nexthop-internet|13.64.0.0/11|Internet|
|mi-13-104-14-nexthop-internet|13.104.0.0/14|Internet|
|mi-20-33-16-nexthop-internet|20.33.0.0/16|Internet|
|mi-20-34-15-nexthop-internet|20.34.0.0/15|Internet|
|mi-20-36-14-nexthop-internet|20.36.0.0/14|Internet|
|mi-20-40-13-nexthop-internet|20.40.0.0/13|Internet|
|mi-20-48-12-nexthop-internet|20.48.0.0/12|Internet|
|mi-20-64-10-nexthop-internet|20.64.0.0/10|Internet|
|mi-20-128-16-nexthop-internet|20.128.0.0/16|Internet|
|mi-20-135-16-nexthop-internet|20.135.0.0/16|Internet|
|mi-20-136-16-nexthop-internet|20.136.0.0/16|Internet|
|mi-20-140-15-nexthop-internet|20.140.0.0/15|Internet|
|mi-20-143-16-nexthop-internet|20.143.0.0/16|Internet|
|mi-20-144-14-nexthop-internet|20.144.0.0/14|Internet|
|mi-20-150-15-nexthop-internet|20.150.0.0/15|Internet|
|mi-20-160-12-nexthop-internet|20.160.0.0/12|Internet|
|mi-20-176-14-nexthop-internet|20.176.0.0/14|Internet|
|mi-20-180-14-nexthop-internet|20.180.0.0/14|Internet|
|mi-20-184-13-nexthop-internet|20.184.0.0/13|Internet|
|mi-20-192-10-nexthop-internet|20.192.0.0/10|Internet|
|mi-40-64-10-nexthop-internet|40.64.0.0/10|Internet|
|mi-51-4-15-nexthop-internet|51.4.0.0/15|Internet|
|mi-51-8-16-nexthop-internet|51.8.0.0/16|Internet|
|mi-51-10-15-nexthop-internet|51.10.0.0/15|Internet|
|mi-51-18-16-nexthop-internet|51.18.0.0/16|Internet|
|mi-51-51-16-nexthop-internet|51.51.0.0/16|Internet|
|mi-51-53-16-nexthop-internet|51.53.0.0/16|Internet|
|mi-51-103-16-nexthop-internet|51.103.0.0/16|Internet|
|mi-51-104-15-nexthop-internet|51.104.0.0/15|Internet|
|mi-51-132-16-nexthop-internet|51.132.0.0/16|Internet|
|mi-51-136-15-nexthop-internet|51.136.0.0/15|Internet|
|mi-51-138-16-nexthop-internet|51.138.0.0/16|Internet|
|mi-51-140-14-nexthop-internet|51.140.0.0/14|Internet|
|mi-51-144-15-nexthop-internet|51.144.0.0/15|Internet|
|mi-52-96-12-nexthop-internet|52.96.0.0/12|Internet|
|mi-52-112-14-nexthop-internet|52.112.0.0/14|Internet|
|mi-52-125-16-nexthop-internet|52.125.0.0/16|Internet|
|mi-52-126-15-nexthop-internet|52.126.0.0/15|Internet|
|mi-52-130-15-nexthop-internet|52.130.0.0/15|Internet|
|mi-52-132-14-nexthop-internet|52.132.0.0/14|Internet|
|mi-52-136-13-nexthop-internet|52.136.0.0/13|Internet|
|mi-52-145-16-nexthop-internet|52.145.0.0/16|Internet|
|mi-52-146-15-nexthop-internet|52.146.0.0/15|Internet|
|mi-52-148-14-nexthop-internet|52.148.0.0/14|Internet|
|mi-52-152-13-nexthop-internet|52.152.0.0/13|Internet|
|mi-52-160-11-nexthop-internet|52.160.0.0/11|Internet|
|mi-52-224-11-nexthop-internet|52.224.0.0/11|Internet|
|mi-64-4-18-nexthop-internet|64.4.0.0/18|Internet|
|mi-65-52-14-nexthop-internet|65.52.0.0/14|Internet|
|mi-66-119-144-20-nexthop-internet|66.119.144.0/20|Internet|
|mi-70-37-17-nexthop-internet|70.37.0.0/17|Internet|
|mi-70-37-128-18-nexthop-internet|70.37.128.0/18|Internet|
|mi-91-190-216-21-nexthop-internet|91.190.216.0/21|Internet|
|mi-94-245-64-18-nexthop-internet|94.245.64.0/18|Internet|
|mi-103-9-8-22-nexthop-internet|103.9.8.0/22|Internet|
|mi-103-25-156-24-nexthop-internet|103.25.156.0/24|Internet|
|mi-103-25-157-24-nexthop-internet|103.25.157.0/24|Internet|
|mi-103-25-158-23-nexthop-internet|103.25.158.0/23|Internet|
|mi-103-36-96-22-nexthop-internet|103.36.96.0/22|Internet|
|mi-103-255-140-22-nexthop-internet|103.255.140.0/22|Internet|
|mi-104-40-13-nexthop-internet|104.40.0.0/13|Internet|
|mi-104-146-15-nexthop-internet|104.146.0.0/15|Internet|
|mi-104-208-13-nexthop-internet|104.208.0.0/13|Internet|
|mi-111-221-16-20-nexthop-internet|111.221.16.0/20|Internet|
|mi-111-221-64-18-nexthop-internet|111.221.64.0/18|Internet|
|mi-129-75-16-nexthop-internet|129.75.0.0/16|Internet|
|mi-131-107-16-nexthop-internet|131.107.0.0/16|Internet|
|mi-131-253-1-24-nexthop-internet|131.253.1.0/24|Internet|
|mi-131-253-3-24-nexthop-internet|131.253.3.0/24|Internet|
|mi-131-253-5-24-nexthop-internet|131.253.5.0/24|Internet|
|mi-131-253-6-24-nexthop-internet|131.253.6.0/24|Internet|
|mi-131-253-8-24-nexthop-internet|131.253.8.0/24|Internet|
|mi-131-253-12-22-nexthop-internet|131.253.12.0/22|Internet|
|mi-131-253-16-23-nexthop-internet|131.253.16.0/23|Internet|
|mi-131-253-18-24-nexthop-internet|131.253.18.0/24|Internet|
|mi-131-253-21-24-nexthop-internet|131.253.21.0/24|Internet|
|mi-131-253-22-23-nexthop-internet|131.253.22.0/23|Internet|
|mi-131-253-24-21-nexthop-internet|131.253.24.0/21|Internet|
|mi-131-253-32-20-nexthop-internet|131.253.32.0/20|Internet|
|mi-131-253-61-24-nexthop-internet|131.253.61.0/24|Internet|
|mi-131-253-62-23-nexthop-internet|131.253.62.0/23|Internet|
|mi-131-253-64-18-nexthop-internet|131.253.64.0/18|Internet|
|mi-131-253-128-17-nexthop-internet|131.253.128.0/17|Internet|
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
|mi-167-105-16-nexthop-internet|167.105.0.0/16|Internet|
|mi-167-220-16-nexthop-internet|167.220.0.0/16|Internet|
|mi-168-61-16-nexthop-internet|168.61.0.0/16|Internet|
|mi-168-62-15-nexthop-internet|168.62.0.0/15|Internet|
|mi-191-232-13-nexthop-internet|191.232.0.0/13|Internet|
|mi-192-32-16-nexthop-internet|192.32.0.0/16|Internet|
|mi-192-48-225-24-nexthop-internet|192.48.225.0/24|Internet|
|mi-192-84-159-24-nexthop-internet|192.84.159.0/24|Internet|
|mi-192-84-160-23-nexthop-internet|192.84.160.0/23|Internet|
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
|mi-23-96-13-nexthop-internet|23.96.0.0/13|Internet|
|mi-42-159-16-nexthop-internet|42.159.0.0/16|Internet|
|mi-51-13-17-nexthop-internet|51.13.0.0/17|Internet|
|mi-51-107-16-nexthop-internet|51.107.0.0/16|Internet|
|mi-51-116-16-nexthop-internet|51.116.0.0/16|Internet|
|mi-51-120-16-nexthop-internet|51.120.0.0/16|Internet|
|mi-51-120-128-17-nexthop-internet|51.120.128.0/17|Internet|
|mi-51-124-16-nexthop-internet|51.124.0.0/16|Internet|
|mi-102-37-18-nexthop-internet|102.37.0.0/18|Internet|
|mi-102-133-16-nexthop-internet|102.133.0.0/16|Internet|
|mi-199-30-16-20-nexthop-internet|199.30.16.0/20|Internet|
|mi-204-79-180-24-nexthop-internet|204.79.180.0/24|Internet|
||||

\* MI SUBNET se refiere al intervalo de direcciones IP de la subred con el formato x.x.x.x/y. Puede encontrar esta información en Portal de Azure, en las propiedades de subred.

Además, puede agregar entradas a la tabla de rutas para enrutar el tráfico que tiene intervalos IP privados locales como destino a través de una puerta de enlace de red virtual o de un dispositivo de red virtual (NVA).

Si la red virtual incluye un DNS personalizado, el servidor DNS personalizado debe ser capaz de resolver los registros DNS públicos. El uso de características adicionales, como la autenticación de Azure AD, puede requerir resolver FQDN adicionales. Para más información, consulte [Configuración de un DNS personalizado ](custom-dns-configure.md).

### <a name="networking-constraints"></a>Restricciones de redes

**TLS 1.2 se aplica en las conexiones salientes**: En enero 2020, Microsoft aplicó TLS 1.2 para el tráfico entre servicios en todos los servicios de Azure. En el caso de Instancia administrada de Azure SQL, esto supuso la aplicación de TLS 1.2 en las conexiones salientes usadas para la replicación y las conexiones del servidor vinculado a SQL Server. Si usa versiones de SQL Server anteriores a 2016 con SQL Managed Instance, asegúrese de que se hayan aplicado las [actualizaciones específicas de TLS 1.2](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server).

Las siguientes características de red virtual no se admiten actualmente con SQL Managed Instance:

- **Emparejamiento de Microsoft**: la habilitación del [emparejamiento de Microsoft](../../expressroute/expressroute-faqs.md#microsoft-peering) en circuitos de ExpressRoute emparejados directamente o de manera transitiva con la red virtual en la que reside SQL Managed Instance afecta al flujo de tráfico entre los componentes de SQL Managed Instance dentro de la red virtual y los servicios de los que depende, causando problemas de disponibilidad. Se prevé que se produzcan errores en las implementaciones de SQL Managed Instance en la red virtual con el emparejamiento de Microsoft habilitado.
- **Emparejamiento global de redes virtuales**: la conectividad de [emparejamiento de redes virtuales](../../virtual-network/virtual-network-peering-overview.md) entre regiones de Azure no funciona con Instancia administrada de SQL debido a las [restricciones del equilibrador de carga documentadas](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers).
- **AzurePlatformDNS**: el uso de la [etiqueta de servicio](../../virtual-network/service-tags-overview.md) AzurePlatformDNS para bloquear la resolución DNS de la plataforma hace que SQL Managed Instance no esté disponible. Aunque SQL Managed Instance admite DNS definida por el cliente para la resolución DNS dentro del motor, existe una dependencia en la DNS de la plataforma para las operaciones de plataforma.
- **NAT Gateway**: el uso de [Azure Virtual Network NAT](../../virtual-network/nat-overview.md) para controlar la conectividad saliente con una dirección IP pública específica, representaría SQL Managed Instance como no disponible. Actualmente, el servicio SQL Managed Instance está limitado al uso de un equilibrador de carga básico que no proporciona coexistencia de flujos entrantes y salientes con Virtual Network NAT.

### <a name="deprecated-network-requirements-without-service-aided-subnet-configuration"></a>[En desuso] Requisitos de red sin configuración de subred asistida por servicio

Implemente SQL Managed Instance en una subred dedicada dentro de la red virtual. La subred debe tener estas características:

- **Subred dedicada:** la subred de SQL Managed Instance no puede contener ningún otro servicio en la nube asociado a ella ni puede ser una subred de puerta de enlace. La subred no puede contener ningún recurso, a excepción de SQL Managed Instance, ni se le pueden agregar posteriormente otros tipos de recursos.
- **Grupo de seguridad de red (NSG):** Un NSG que esté asociado a la red virtual debe definir [reglas de seguridad de entrada](#mandatory-inbound-security-rules) y [reglas de seguridad de salida](#mandatory-outbound-security-rules) antes que cualquier otras reglas. Puede usar un NSG para controlar el acceso al punto de conexión de datos de SQL Managed Instance mediante el filtrado del tráfico en el puerto 1433 y en los puertos 11000 a 11999 cuando SQL Managed Instance se configura para conexiones de redirección.
- **Tabla de rutas definida por el usuario (UDR):** una tabla UDR que esté asociada a la red virtual debe incluir [entradas](#user-defined-routes) específicas.
- **Sin puntos de conexión de servicio:** no debe haber ningún punto de conexión de servicio asociado con la subred de SQL Managed Instance. Asegúrese de que la opción de puntos de conexión de servicio esté deshabilitada al crear la red virtual.
- **Suficientes direcciones IP:** la subred de Instancia administrada de SQL debe tener al menos 16 direcciones IP. El mínimo recomendado es 32 direcciones IP. Para obtener más información, consulte [Determinación del tamaño de la subred de SQL Managed Instance](vnet-subnet-determine-size.md). Puede implementar instancias administradas en [la red existente](vnet-existing-add-subnet.md) después de configurarla para satisfacer [los requisitos de red de SQL Managed Instance](#network-requirements). De lo contrario, cree [una red y una subred](virtual-network-subnet-create-arm-template.md).

> [!IMPORTANT]
> No se puede implementar una nueva instancia administrada si la subred de destino no tiene estas características. Cuando se crea una instancia administrada, se aplica una directiva de intención de red en la subred para evitar cambios no compatibles con la configuración de red. Después de quitar la última instancia de la subred, también se quitará la directiva de intención de red.

### <a name="mandatory-inbound-security-rules"></a>Reglas de seguridad de entrada obligatorias

| Nombre       |Port                        |Protocolo|Source           |Destination|Acción|
|------------|----------------------------|--------|-----------------|-----------|------|
|management  |9000, 9003, 1438, 1440, 1452|TCP     |Any              |MI SUBNET  |Allow |
|mi_subnet   |Any                         |Any     |MI SUBNET        |MI SUBNET  |Allow |
|health_probe|Any                         |Any     |AzureLoadBalancer|MI SUBNET  |Allow |

### <a name="mandatory-outbound-security-rules"></a>Reglas de seguridad de salida obligatorias

| Nombre       |Port          |Protocolo|Source           |Destination|Acción|
|------------|--------------|--------|-----------------|-----------|------|
|management  |443, 12000    |TCP     |MI SUBNET        |AzureCloud |Allow |
|mi_subnet   |Any           |Any     |MI SUBNET        |MI SUBNET  |Allow |

> [!IMPORTANT]
> Asegúrese de que solo haya una regla de entrada para los puertos 9000, 9003, 1438, 1440 y 1452, y una regla de salida para los puertos 443 y 12000. El aprovisionamiento de Instancia administrada de SQL mediante implementaciones de Azure Resource Manager genera un error si las reglas de entrada y salida están configuradas por separado para cada puerto. Si estos puertos están en reglas distintas, la implementación generará el código de error `VnetSubnetConflictWithIntendedPolicy`.

\* MI SUBNET se refiere al intervalo de direcciones IP de la subred con el formato x.x.x.x/y. Puede encontrar esta información en Portal de Azure, en las propiedades de subred.

> [!IMPORTANT]
> Aunque las reglas de seguridad de entrada obligatorias permiten el tráfico desde _cualquier_ origen en los puertos 9000, 9003, 1438, 1440 y 1452, estos puertos están protegidos por un firewall integrado. Para más información, consulte [Determinación de la dirección del punto de conexión de administración](management-endpoint-find-ip-address.md).

> [!NOTE]
> Si usa replicación transaccional en SQL Managed Instance y emplea cualquier base de datos de instancias como publicador o distribuidor, abra el puerto 445 (salida TCP) en las reglas de seguridad de la subred. Este puerto permitirá el acceso al recurso compartido de archivos de Azure.

### <a name="user-defined-routes"></a>rutas definidas por el usuario

|Nombre|Prefijo de dirección|Próximo salto|
|----|--------------|-------|
|subnet_to_vnetlocal|MI SUBNET|Virtual network|
|mi-13-64-11-nexthop-internet|13.64.0.0/11|Internet|
|mi-13-104-14-nexthop-internet|13.104.0.0/14|Internet|
|mi-20-33-16-nexthop-internet|20.33.0.0/16|Internet|
|mi-20-34-15-nexthop-internet|20.34.0.0/15|Internet|
|mi-20-36-14-nexthop-internet|20.36.0.0/14|Internet|
|mi-20-40-13-nexthop-internet|20.40.0.0/13|Internet|
|mi-20-48-12-nexthop-internet|20.48.0.0/12|Internet|
|mi-20-64-10-nexthop-internet|20.64.0.0/10|Internet|
|mi-20-128-16-nexthop-internet|20.128.0.0/16|Internet|
|mi-20-135-16-nexthop-internet|20.135.0.0/16|Internet|
|mi-20-136-16-nexthop-internet|20.136.0.0/16|Internet|
|mi-20-140-15-nexthop-internet|20.140.0.0/15|Internet|
|mi-20-143-16-nexthop-internet|20.143.0.0/16|Internet|
|mi-20-144-14-nexthop-internet|20.144.0.0/14|Internet|
|mi-20-150-15-nexthop-internet|20.150.0.0/15|Internet|
|mi-20-160-12-nexthop-internet|20.160.0.0/12|Internet|
|mi-20-176-14-nexthop-internet|20.176.0.0/14|Internet|
|mi-20-180-14-nexthop-internet|20.180.0.0/14|Internet|
|mi-20-184-13-nexthop-internet|20.184.0.0/13|Internet|
|mi-20-192-10-nexthop-internet|20.192.0.0/10|Internet|
|mi-40-64-10-nexthop-internet|40.64.0.0/10|Internet|
|mi-51-4-15-nexthop-internet|51.4.0.0/15|Internet|
|mi-51-8-16-nexthop-internet|51.8.0.0/16|Internet|
|mi-51-10-15-nexthop-internet|51.10.0.0/15|Internet|
|mi-51-18-16-nexthop-internet|51.18.0.0/16|Internet|
|mi-51-51-16-nexthop-internet|51.51.0.0/16|Internet|
|mi-51-53-16-nexthop-internet|51.53.0.0/16|Internet|
|mi-51-103-16-nexthop-internet|51.103.0.0/16|Internet|
|mi-51-104-15-nexthop-internet|51.104.0.0/15|Internet|
|mi-51-132-16-nexthop-internet|51.132.0.0/16|Internet|
|mi-51-136-15-nexthop-internet|51.136.0.0/15|Internet|
|mi-51-138-16-nexthop-internet|51.138.0.0/16|Internet|
|mi-51-140-14-nexthop-internet|51.140.0.0/14|Internet|
|mi-51-144-15-nexthop-internet|51.144.0.0/15|Internet|
|mi-52-96-12-nexthop-internet|52.96.0.0/12|Internet|
|mi-52-112-14-nexthop-internet|52.112.0.0/14|Internet|
|mi-52-125-16-nexthop-internet|52.125.0.0/16|Internet|
|mi-52-126-15-nexthop-internet|52.126.0.0/15|Internet|
|mi-52-130-15-nexthop-internet|52.130.0.0/15|Internet|
|mi-52-132-14-nexthop-internet|52.132.0.0/14|Internet|
|mi-52-136-13-nexthop-internet|52.136.0.0/13|Internet|
|mi-52-145-16-nexthop-internet|52.145.0.0/16|Internet|
|mi-52-146-15-nexthop-internet|52.146.0.0/15|Internet|
|mi-52-148-14-nexthop-internet|52.148.0.0/14|Internet|
|mi-52-152-13-nexthop-internet|52.152.0.0/13|Internet|
|mi-52-160-11-nexthop-internet|52.160.0.0/11|Internet|
|mi-52-224-11-nexthop-internet|52.224.0.0/11|Internet|
|mi-64-4-18-nexthop-internet|64.4.0.0/18|Internet|
|mi-65-52-14-nexthop-internet|65.52.0.0/14|Internet|
|mi-66-119-144-20-nexthop-internet|66.119.144.0/20|Internet|
|mi-70-37-17-nexthop-internet|70.37.0.0/17|Internet|
|mi-70-37-128-18-nexthop-internet|70.37.128.0/18|Internet|
|mi-91-190-216-21-nexthop-internet|91.190.216.0/21|Internet|
|mi-94-245-64-18-nexthop-internet|94.245.64.0/18|Internet|
|mi-103-9-8-22-nexthop-internet|103.9.8.0/22|Internet|
|mi-103-25-156-24-nexthop-internet|103.25.156.0/24|Internet|
|mi-103-25-157-24-nexthop-internet|103.25.157.0/24|Internet|
|mi-103-25-158-23-nexthop-internet|103.25.158.0/23|Internet|
|mi-103-36-96-22-nexthop-internet|103.36.96.0/22|Internet|
|mi-103-255-140-22-nexthop-internet|103.255.140.0/22|Internet|
|mi-104-40-13-nexthop-internet|104.40.0.0/13|Internet|
|mi-104-146-15-nexthop-internet|104.146.0.0/15|Internet|
|mi-104-208-13-nexthop-internet|104.208.0.0/13|Internet|
|mi-111-221-16-20-nexthop-internet|111.221.16.0/20|Internet|
|mi-111-221-64-18-nexthop-internet|111.221.64.0/18|Internet|
|mi-129-75-16-nexthop-internet|129.75.0.0/16|Internet|
|mi-131-107-16-nexthop-internet|131.107.0.0/16|Internet|
|mi-131-253-1-24-nexthop-internet|131.253.1.0/24|Internet|
|mi-131-253-3-24-nexthop-internet|131.253.3.0/24|Internet|
|mi-131-253-5-24-nexthop-internet|131.253.5.0/24|Internet|
|mi-131-253-6-24-nexthop-internet|131.253.6.0/24|Internet|
|mi-131-253-8-24-nexthop-internet|131.253.8.0/24|Internet|
|mi-131-253-12-22-nexthop-internet|131.253.12.0/22|Internet|
|mi-131-253-16-23-nexthop-internet|131.253.16.0/23|Internet|
|mi-131-253-18-24-nexthop-internet|131.253.18.0/24|Internet|
|mi-131-253-21-24-nexthop-internet|131.253.21.0/24|Internet|
|mi-131-253-22-23-nexthop-internet|131.253.22.0/23|Internet|
|mi-131-253-24-21-nexthop-internet|131.253.24.0/21|Internet|
|mi-131-253-32-20-nexthop-internet|131.253.32.0/20|Internet|
|mi-131-253-61-24-nexthop-internet|131.253.61.0/24|Internet|
|mi-131-253-62-23-nexthop-internet|131.253.62.0/23|Internet|
|mi-131-253-64-18-nexthop-internet|131.253.64.0/18|Internet|
|mi-131-253-128-17-nexthop-internet|131.253.128.0/17|Internet|
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
|mi-167-105-16-nexthop-internet|167.105.0.0/16|Internet|
|mi-167-220-16-nexthop-internet|167.220.0.0/16|Internet|
|mi-168-61-16-nexthop-internet|168.61.0.0/16|Internet|
|mi-168-62-15-nexthop-internet|168.62.0.0/15|Internet|
|mi-191-232-13-nexthop-internet|191.232.0.0/13|Internet|
|mi-192-32-16-nexthop-internet|192.32.0.0/16|Internet|
|mi-192-48-225-24-nexthop-internet|192.48.225.0/24|Internet|
|mi-192-84-159-24-nexthop-internet|192.84.159.0/24|Internet|
|mi-192-84-160-23-nexthop-internet|192.84.160.0/23|Internet|
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
|mi-23-96-13-nexthop-internet|23.96.0.0/13|Internet|
|mi-42-159-16-nexthop-internet|42.159.0.0/16|Internet|
|mi-51-13-17-nexthop-internet|51.13.0.0/17|Internet|
|mi-51-107-16-nexthop-internet|51.107.0.0/16|Internet|
|mi-51-116-16-nexthop-internet|51.116.0.0/16|Internet|
|mi-51-120-16-nexthop-internet|51.120.0.0/16|Internet|
|mi-51-120-128-17-nexthop-internet|51.120.128.0/17|Internet|
|mi-51-124-16-nexthop-internet|51.124.0.0/16|Internet|
|mi-102-37-18-nexthop-internet|102.37.0.0/18|Internet|
|mi-102-133-16-nexthop-internet|102.133.0.0/16|Internet|
|mi-199-30-16-20-nexthop-internet|199.30.16.0/20|Internet|
|mi-204-79-180-24-nexthop-internet|204.79.180.0/24|Internet|
||||

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general, vea  [¿Qué es Instancia administrada de Azure SQL?](sql-managed-instance-paas-overview.md)
- Aprenda a [configurar una nueva red virtual de Azure](virtual-network-subnet-create-arm-template.md) o una [red virtual de Azure existente](vnet-existing-add-subnet.md) donde pueda implementar SQL Managed Instance.
- [Calcule el tamaño de la subred](vnet-subnet-determine-size.md) donde quiere implementar SQL Managed Instance.
- Aprenda a crear una instancia administrada:
  - Desde [Azure Portal](instance-create-quickstart.md).
  - Mediante [PowerShell](scripts/create-configure-managed-instance-powershell.md).
  - Mediante [una plantilla de Azure Resource Manager](https://azure.microsoft.com/resources/templates/101-sqlmi-new-vnet/).
  - Mediante [una plantilla de Azure Resource Manager (JumpBox con SSMS incluido)](https://azure.microsoft.com/resources/templates/201-sqlmi-new-vnet-w-jumpbox/).
