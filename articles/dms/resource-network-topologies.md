---
title: Topologías de red para migraciones de Instancia administrada de SQL
titleSuffix: Azure Database Migration Service
description: Obtenga información sobre las configuraciones de origen y destino para migrar Instancia administrada de Azure SQL mediante Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: article
ms.date: 01/08/2020
ms.openlocfilehash: 69926671730e41845cd28df3108ec86b24a57075
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448518"
---
# <a name="network-topologies-for-azure-sql-managed-instance-migrations-using-azure-database-migration-service"></a>Topologías de red para migraciones de Instancia administrada de Azure SQL mediante Azure Database Migration Service

En este artículo se tratan diversas topologías de red con las que Azure Database Migration Service puede trabajar para proporcionar una experiencia de migración completa desde instancias de SQL Server a Instancia administrada de Azure SQL.

## <a name="azure-sql-managed-instance-configured-for-hybrid-workloads"></a>Instancia administrada de Azure SQL configurada para cargas de trabajo híbridas 

Use esta topología si su Instancia administrada de Azure SQL está conectada a la red local. Este enfoque proporciona el enrutamiento de red más simplificado y genera el máximo rendimiento de los datos durante la migración.

![Topología de red para cargas de trabajo híbridas](media/resource-network-topologies/hybrid-workloads.png)

**Requisitos**

- En este escenario, Instancia administrada de SQL y la instancia de Azure Database Migration Service se crean en la misma instancia de Microsoft Azure Virtual Network, pero usan diferentes subredes.  
- La red virtual que se utiliza en este escenario también está conectada a la red local mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways).

## <a name="sql-managed-instance-isolated-from-the-on-premises-network"></a>Instancia administrada de SQL aislada de la red local

Use esta topología de red si su entorno requiere uno o varios de los siguientes escenarios:

- Instancia administrada de SQL está aislada de la conectividad local pero la instancia de Azure Database Migration Service está conectada a la red local.
- Si se han implementado directivas de control de acceso basado en rol de Azure (RBAC de Azure) y tiene que limitar el acceso de los usuarios a la misma suscripción que hospeda SQL Managed Instance.
- Las redes virtuales que se usan para Instancia administrada de SQL y Azure Database Migration Service se encuentran en suscripciones distintas.

![Topología de red para Instancia administrada aislada de la red local](media/resource-network-topologies/mi-isolated-workload.png)

**Requisitos**

- La red virtual que utiliza Azure Database Migration Service para este escenario también debe estar conectada a la red local mediante (https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways).
- Configure el [emparejamiento de red virtual](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview) entre la red virtual que se usa para Instancia administrada de SQL y Azure Database Migration Service.

## <a name="cloud-to-cloud-migrations-shared-virtual-network"></a>Migraciones de nube a nube: red virtual compartida

Use esta topología si el servidor SQL Server de origen se hospeda en una VM de Azure y comparte la misma red virtual con Instancia administrada de SQL y Azure Database Migration Service.

![Topología de red para migraciones de nube a nube con una red virtual compartida](media/resource-network-topologies/cloud-to-cloud.png)

**Requisitos**

- No hay requisitos adicionales.

## <a name="cloud-to-cloud-migrations-isolated-virtual-network"></a>Migraciones de nube a nube: red virtual aislada

Use esta topología de red si su entorno requiere uno o varios de los siguientes escenarios:

- Instancia administrada de SQL se aprovisiona en una red virtual aislada.
- Si se han implementado directivas de control de acceso basado en rol de Azure (RBAC de Azure) y tiene que limitar el acceso de los usuarios a la misma suscripción que hospeda SQL Managed Instance.
- Las redes virtuales que se usan para Instancia administrada de SQL y Azure Database Migration Service se encuentran en suscripciones distintas.

![Topología de red para migraciones de nube a nube con una red virtual aislada](media/resource-network-topologies/cloud-to-cloud-isolated.png)

**Requisitos**

- Configure el [emparejamiento de red virtual](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview) entre la red virtual que se usa para Instancia administrada de SQL y Azure Database Migration Service.

## <a name="inbound-security-rules"></a>Reglas de seguridad de entrada

| **NOMBRE**   | **PUERTO** | **PROTOCOLO** | **ORIGEN** | **DESTINO** | **ACCIÓN** |
|------------|----------|--------------|------------|-----------------|------------|
| DMS_subnet | Any      | Any          | SUBRED DMS | Any             | Allow      |

## <a name="outbound-security-rules"></a>Reglas de seguridad de entrada

| **NOMBRE**                  | **PUERTO**                                              | **PROTOCOLO** | **ORIGEN** | **DESTINO**           | **ACCIÓN** | **Motivo de la regla**                                                                                                                                                                              |
|---------------------------|-------------------------------------------------------|--------------|------------|---------------------------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| management                | 443,9354                                              | TCP          | Any        | Any                       | Allow      | Comunicación de planos de administración mediante Service Bus y Azure Blob Storage. <br/>(Si está habilitado el emparejamiento de Microsoft, es posible que no necesite esta regla).                                                             |
| Diagnóstico               | 12000                                                 | TCP          | Any        | Any                       | Allow      | DMS usa esta regla para recopilar información de diagnóstico para solucionar problemas.                                                                                                                      |
| Servidor de origen SQL         | 1433 (o puerto TCP IP en el que SQL Server escucha) | TCP          | Any        | Espacio de direcciones local | Allow      | Conectividad del origen de SQL Server desde DMS <br/>(Si dispone de conectividad de sitio a sitio, es posible que no necesite esta regla).                                                                                       |
| Instancia con nombre de SQL Server | 1434                                                  | UDP          | Any        | Espacio de direcciones local | Allow      | Conectividad del origen de las instancias con nombre desde DMS <br/>(Si dispone de conectividad de sitio a sitio, es posible que no necesite esta regla).                                                                        |
| Recurso compartido de SMB                 | 445                                                   | TCP          | Any        | Espacio de direcciones local | Allow      | Recurso compartido de red SMB para DMS para almacenar los archivos de copia de seguridad de base de datos de las migraciones a Azure SQL Database MI y servidores SQL Server en Azure VM <br/>(Si dispone de conectividad de sitio a sitio, es posible que no necesite esta regla). |
| DMS_subnet                | Any                                                   | Any          | Any        | DMS_Subnet                | Allow      |                                                                                                                                                                                                  |

## <a name="see-also"></a>Consulte también

- [Migración de SQL Server a Instancia administrada de SQL](https://docs.microsoft.com/azure/dms/tutorial-sql-server-to-managed-instance)
- [Introducción a los requisitos previos para usar Azure Database Migration Service](https://docs.microsoft.com/azure/dms/pre-reqs)
- [Creación de una red virtual mediante Azure Portal](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general de Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](dms-overview.md).
- Para obtener información actualizada acerca de la disponibilidad regional de Azure Database Migration Service, consulte la página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=database-migration).
