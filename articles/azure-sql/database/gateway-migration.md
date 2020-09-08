---
title: Aviso de migración de tráfico de puertas de enlace
description: El artículo proporciona un aviso a los usuarios sobre la migración de direcciones IP de puertas de enlace de Azure SQL Database
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: sqldbrb=1 
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: vanto
ms.date: 07/01/2019
ms.openlocfilehash: af5486630eb89de198b6ed2975a919b04b01a902
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88507525"
---
# <a name="azure-sql-database-traffic-migration-to-newer-gateways"></a>Migración de tráfico de Azure SQL Database a puertas de enlace más recientes
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

A media que la infraestructura de Azure mejora, Microsoft actualizará periódicamente el hardware para asegurarse de que ofrecemos la mejor experiencia de cliente posible. En los próximos meses, tenemos previsto agregar puertas de enlace basadas en generaciones de hardware más recientes, migrar el tráfico a ellas y retirar puertas de enlace basadas en hardware más antiguo en algunas regiones.  

Se enviará un aviso a los clientes por correo electrónico y en Azure Portal con antelación de cualquier cambio en las puertas de enlace disponibles en cada región. La información más actualizada se mantendrá en la tabla [Direcciones IP de la puerta de enlace de Azure SQL Database](connectivity-architecture.md#gateway-ip-addresses).

## <a name="status-updates"></a>Actualizaciones de estado

# <a name="in-progress"></a>[En curso](#tab/in-progress-ip)
### <a name="september-2020"></a>Septiembre de 2020
Se están agregando nuevas puertas de enlace de SQL a las siguientes regiones. Estas puertas de enlace de SQL comenzarán a aceptar el tráfico del cliente a partir del **15 de septiembre de 2020**:

- Sudeste de Australia: 13.77.48.10
- Este de Canadá: 40.86.226.166, 52.242.30.154
- Sur de Reino Unido: 51.140.184.11, 51.105.64.0

Las puertas de enlace de SQL existentes comenzarán a aceptar tráfico en las siguientes regiones. Estas puertas de enlace de SQL comenzarán a aceptar el tráfico del cliente a partir del **15 de septiembre de 2020**:

- Sudeste de Australia: 191.239.192.109 y 13.73.109.251
- Centro de EE. UU.: 13.67.215.62, 52.182.137.15, 23.99.160.139, 104.208.16.96 y 104.208.21.1
- Este de Asia: 191.234.2.139, 52.175.33.150 y 13.75.32.4
- Este de EE. UU.: 40.121.158.30, 40.79.153.12, 191.238.6.43 y 40.78.225.32
- Este de EE. UU. 2: 40.79.84.180, 52.177.185.181, 52.167.104.0, 191.239.224.107 y 104.208.150.3
- Centro de Francia: 40.79.137.0 y 40.79.129.1
- Japón Occidental: 104.214.148.156, 40.74.100.192, 191.238.68.11 y 40.74.97.10
- Centro-norte de EE. UU.: 23.96.178.199, 23.98.55.75 y 52.162.104.33
- Sudeste de Asia: 104.43.15.0, 23.100.117.95 y 40.78.232.3
- Oeste de EE. UU.: 104.42.238.205, 23.99.34.75 y 13.86.216.196

Se están agregando nuevas puertas de enlace de SQL a las siguientes regiones. Estas puertas de enlace de SQL comenzarán a aceptar el tráfico del cliente a partir del **10 de septiembre de 2020**:

- Centro-oeste de EE. UU.: 13.78.248.43 
- Norte de Sudáfrica: 102.133.120.2  

Se están agregando nuevas puertas de enlace de SQL a las siguientes regiones. Estas puertas de enlace de SQL comenzarán a aceptar el tráfico del cliente a partir del **1 de septiembre de 2020**:

- Norte de Europa: 13.74.104.113 
- Oeste de EE. UU. 2: 40.78.248.10 
- Oeste de Europa: 52.236.184.163 
- Centro y Sur de EE. UU.: 20.45.121.1, 20.49.88.1 

Las puertas de enlace de SQL existentes comenzarán a aceptar tráfico en las siguientes regiones. Estas puertas de enlace de SQL comenzarán a aceptar el tráfico del cliente a partir del **1 de septiembre de 2020**:
- Este de Japón: 40.79.184.8, 40.79.192.5

# <a name="completed"></a>[Completado](#tab/completed-ip)

Se han completado las siguientes migraciones de puerta de enlace: 

### <a name="august-2020"></a>Agosto de 2020

Se están agregando nuevas puertas de enlace de SQL a las siguientes regiones:

- Este de Australia: 13.70.112.9
- Centro de Canadá: 52.246.152.0, 20.38.144.1 
- Oeste de EE. UU. 2: 40.78.240.8

Estas puertas de enlace de SQL comenzarán a aceptar el tráfico del cliente el 10 de agosto de 2020. 

### <a name="october-2019"></a>Octubre de 2019
- Sur de Brasil
- Oeste de EE. UU.
- Oeste de Europa
- Este de EE. UU.
- Centro de EE. UU.
- Sudeste de Asia
- Centro-sur de EE. UU.
- Norte de Europa
- Centro-Norte de EE. UU
- Japón Occidental
- Japón Oriental
- Este de EE. UU. 2
- Este de Asia

---

## <a name="impact-of-this-change"></a>Impacto de este cambio

La migración del tráfico puede cambiar la dirección IP pública que DNS resuelve para su base de datos en Azure SQL Database.
Puede verse afectado si:

- Ha codificado de forma rígida la dirección IP de una puerta de enlace determinada en el firewall local.
- Tiene subredes que usan Microsoft.SQL como punto de conexión de servicio, pero no se puede comunicar con las direcciones IP de puerta de enlace.
- Usa la [configuración con redundancia de zona](high-availability-sla.md#zone-redundant-configuration) para la base de datos

No se verá afectado en los siguientes casos:
 
- La directiva de conexión tiene redireccionamiento.
- Tiene conexiones a SQL Database desde Azure y con etiquetas de servicio.
- Las conexiones realizadas mediante versiones compatibles de JDBC Driver para SQL Server no percibirán ningún impacto. Para las versiones compatibles de JDBC, consulte [Descarga de Microsoft JDBC Driver para SQL Server](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server).

## <a name="what-to-do-you-do-if-youre-affected"></a>Qué puede hacer si se ve afectado

Es recomendable que permita el tráfico saliente a direcciones IP para todas las [direcciones IP de puerta de enlace](connectivity-architecture.md#gateway-ip-addresses) de la región en el puerto TCP 1433 y el intervalo de puertos de 11000 a 11999. Esta recomendación se aplica a los clientes que se conectan desde el entorno local y también a los que se conectan a través de puntos de conexión de servicio. Para obtener más información sobre los intervalos de puertos, consulte [Directiva de conexión](connectivity-architecture.md#connection-policy).

Es posible que las conexiones realizadas desde aplicaciones que usan versiones de Microsoft JDBC Driver anteriores a 4.0 no superen la validación de certificados. Las versiones inferiores de Microsoft JDBC dependen del nombre común (CN) en el campo Asunto del certificado. La mitigación consiste en asegurarse de que la propiedad hostNameInCertificate esté establecida en *.database.windows.net. Para más información sobre cómo establecer la propiedad hostNameInCertificate, consulte [Conexión con el cifrado](/sql/connect/jdbc/connecting-with-ssl-encryption).

Si la mitigación anterior no funciona, envíe una solicitud de soporte técnico para SQL Database o Instancia administrada de SQL con la dirección URL siguiente: https://aka.ms/getazuresupport

## <a name="next-steps"></a>Pasos siguientes

- Obtener más información sobre la [Arquitectura de conectividad de Azure SQL](connectivity-architecture.md)
