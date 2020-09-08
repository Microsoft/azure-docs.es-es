---
title: Configuración y administración de una referencia de contenido
titleSuffix: Azure SQL Managed Instance
description: Guía de referencia de contenido que le enseña a configurar y administrar Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, carlr
ms.date: 04/16/2019
ms.openlocfilehash: de8d1b7aba7763799228c831af61bf83a79937bb
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167987"
---
# <a name="azure-sql-managed-instance-content-reference"></a>Referencia de contenido de Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo puede encontrar una referencia de contenido a varias guías, scripts y explicaciones que le ayudan a administrar y configurar Instancia administrada de Azure SQL.

## <a name="load-data"></a>Cargar datos

- [Migración a Instancia administrada de Azure SQL](migrate-to-instance-from-sql-server.md): conozca el proceso de migración y las herramientas que se recomiendan para la migración a Instancia administrada de Azure SQL.
- [Migración del certificado de TDE a Instancia administrada de Azure SQL](tde-certificate-migrate.md): si su base de datos de SQL Server está protegida con cifrado de datos transparente (TDE), tendría que migrar el certificado que Instancia administrada de SQL pueda usar para descifrar la copia de seguridad que quiera restaurar en Azure.
- [Importación de una base de datos desde un archivo BACPAC](../database/database-import.md)
- [Exportación de una base de datos a un archivo BACPAC](../database/database-export.md)
- [Carga de datos con BCP](../load-from-csv-with-bcp.md)
- [Carga de datos con Azure Data Factory](../../data-factory/connector-azure-sql-database.md?toc=/azure/sql-database/toc.json)

## <a name="network-configuration"></a>Network configuration (Configuración de red)

- [Determinación del tamaño de la subred](vnet-subnet-determine-size.md) puesto que no se puede cambiar el tamaño de la subred después de implementar Instancia administrada de SQL, debe calcular qué intervalo de direcciones IP se necesita según el número y los tipos de instancias administradas que planee implementar en la subred. 
- [Creación de una red virtual y una subred](virtual-network-subnet-create-arm-template.md): configure la red virtual y la subred según los [requisitos de red](connectivity-architecture-overview.md#network-requirements). 
- [Configuración de una red virtual y una subred existentes](vnet-existing-add-subnet.md): compruebe los requisitos de red y configure la red virtual y la subred existentes para implementar Instancia administrada de SQL. 
- [Configuración de DNS personalizado](custom-dns-configure.md): configure un DNS personalizado para conceder acceso a recursos externos a dominios personalizados desde Instancia administrada de SQL a través de un servidor vinculado de perfiles de correo de base de datos. 
- [Sincronización de la configuración de red](azure-app-sync-network-configuration.md): actualice el plan de configuración de red si no puede establecer una conexión después de [integrar la aplicación con una red virtual de Azure](../../app-service/web-sites-integrate-with-vnet.md).
- [Búsqueda de la dirección IP del punto de conexión de administración](management-endpoint-find-ip-address.md): determine el punto de conexión público que usa Instancia administrada de SQL con fines de administración. 
- [Comprobación de la protección de firewall integrada](management-endpoint-verify-built-in-firewall.md): compruebe que Instancia administrada de SQL solo permita el tráfico en los puertos necesarios, y otras reglas de firewall integradas. 
- [Conexión de aplicaciones](connect-application-instance.md): conozca los distintos patrones para conectar las aplicaciones a Instancia administrada de SQL.

## <a name="feature-configuration"></a>Configuración de características

- [Configuración de la autenticación de Azure AD](../database/authentication-aad-configure.md)
- [Configuración del acceso condicional](../database/conditional-access-configure.md)
- [Autenticación multifactor de Azure AD](../database/authentication-mfa-ssms-overview.md)
- [Configuración de Multi-factor Authentication](../database/authentication-mfa-ssms-configure.md)
- [Configuración de una directiva de retención temporal](../database/temporal-tables-retention-policy.md)
- [Configuración de TDE con BYOK](../database/transparent-data-encryption-byok-configure.md)
- [Rotación de claves BYOK de TDE](../database/transparent-data-encryption-byok-key-rotation.md)
- [Eliminación de un protector de TDE](../database/transparent-data-encryption-byok-remove-tde-protector.md)
- [Configuración de OLTP en memoria](../in-memory-oltp-configure.md)
- [Configuración de Azure Automation](../database/automation-manage.md)
- La [replicación transaccional](replication-between-two-instances-configure-tutorial.md) le permite replicar los datos entre instancias administradas, o desde un entorno local de SQL Server hasta Instancia administrada de SQL y viceversa.
- [Configuración de la detección de amenazas](threat-detection-configure.md):la [detección de amenazas](../database/threat-detection-overview.md) es una característica integrada de Instancia administrada de Azure SQL que detecta diversos ataques potenciales, como la inyección de código SQL o el acceso desde ubicaciones sospechosas. 
- La [creación de alertas](alerts-create.md) permite configurar alertas de las métricas supervisadas, como el uso de CPU, el consumo de espacio de almacenamiento, el número de IOPS y demás para Instancia administrada de SQL. 

## <a name="monitoring-and-tuning"></a>Supervisión y ajuste

- [Ajuste manual](../database/performance-guidance.md)
- [Uso de DMV para supervisar el rendimiento](../database/monitoring-with-dmvs.md)
- [Uso del Almacén de consultas para supervisar el rendimiento](https://docs.microsoft.com/sql/relational-databases/performance/best-practice-with-the-query-store#Insight)
- [Solución de problemas de rendimiento con Intelligent Insights](../database/intelligent-insights-troubleshoot-performance.md)
- [Uso del registro de diagnóstico de Intelligent Insights](../database/intelligent-insights-use-diagnostics-log.md)
- [Supervisión del espacio OLTP en memoria](../in-memory-oltp-monitor-space.md)

### <a name="extended-events"></a>Eventos extendidos

- [Eventos extendidos](../database/xevent-db-diff-from-svr.md)
- [Almacenamiento de eventos extendidos en el archivo de evento](../database/xevent-code-event-file.md)
- [Almacenamiento de eventos extendidos en el búfer en anillo](../database/xevent-code-ring-buffer.md)

### <a name="alerting"></a>Alertas

- [Creación de alertas en una instancia administrada](alerts-create.md)

## <a name="operations"></a>Operaciones

- [Conmutación por error manual iniciada por el usuario en SQL Managed Instance](user-initiated-failover.md)

## <a name="develop-applications"></a>Desarrollo de aplicaciones

- [Conectividad](../database/connect-query-content-reference-guide.md#libraries)
- [Uso de Spark Connector](../../cosmos-db/spark-connector.md)
- [Autenticación de una aplicación](../database/application-authentication-get-client-id-keys.md)
- [Uso del procesamiento por lotes para mejorar el rendimiento](../performance-improve-use-batching.md)
- [Guía de conectividad](../database/troubleshoot-common-connectivity-issues.md)
- [Alias DNS](../database/dns-alias-overview.md)
- [Configuración de un alias DNS mediante PowerShell](../database/dns-alias-powershell-create.md)
- [Puertos: ADO.NET](../database/adonet-v12-develop-direct-route-ports.md)
- [C y C ++](../database/develop-cplusplus-simple.md)
- [Excel](../database/connect-excel.md)

## <a name="design-applications"></a>Diseño de aplicaciones

- [Diseño de la recuperación ante desastres](../database/designing-cloud-solutions-for-disaster-recovery.md)
- [Diseño para grupos elásticos](../database/disaster-recovery-strategies-for-applications-with-elastic-pool.md)
- [Diseño para actualización de aplicaciones](../database/manage-application-rolling-upgrade.md)

### <a name="design-multi-tenant-saas-applications"></a>Diseño de aplicaciones SaaS multiinquilino

- [Patrones de diseño SaaS](../database/saas-tenancy-app-design-patterns.md)
- [Video Indexer SaaS](../database/saas-tenancy-video-index-wingtip-brk3120-20171011.md)
- [Seguridad de aplicaciones SaaS](../database/saas-tenancy-elastic-tools-multi-tenant-row-level-security.md)

## <a name="next-steps"></a>Pasos siguientes

Comience por [implementar Instancia administrada de SQL](instance-create-quickstart.md).
