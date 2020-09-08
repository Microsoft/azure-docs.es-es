---
title: Arquitectura de conectividad de Azure SQL Database
description: En este documento se explica la arquitectura de conectividad de Azure SQL Database para las conexiones de bases de datos desde dentro o desde fuera de Azure.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: fasttrack-edit, sqldbrb=1
titleSuffix: Azure SQL Database and Azure Synapse Analytics
ms.devlang: ''
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: carlrab, vanto
ms.date: 06/26/2020
ms.openlocfilehash: 4d48ca3685dca36157307e7cb4b3d25261c243aa
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88705748"
---
# <a name="azure-sql-database-and-azure-synapse-analytics-connectivity-architecture"></a>Arquitectura de conectividad de Azure SQL Database y Azure Synapse Analytics
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

En este artículo se explica la arquitectura de varios componentes que dirigen el tráfico de red a un servidor en Azure SQL Database o Azure Synapse Analytics. También se explican distintas directivas de conexión y cómo afectan a los clientes que se conectan desde dentro de Azure y a los clientes que se conectan desde fuera de Azure.

> [!IMPORTANT]
> Este artículo *no* se aplica a **Instancia administrada de Azure SQL**. Consulte [Arquitectura de conectividad para una instancia administrada](../managed-instance/connectivity-architecture-overview.md).

## <a name="connectivity-architecture"></a>Arquitectura de conectividad

En este diagrama se proporciona una descripción general de la arquitectura de conectividad.

![Descripción general de la arquitectura](./media/connectivity-architecture/connectivity-overview.png)

En los pasos siguientes se describe cómo se establece una conexión a Azure SQL Database:

- Los clientes se conectan a una puerta de enlace, que tiene una dirección IP pública y escucha en el puerto 1433.
- La puerta de enlace, dependiendo de la directiva de conexión efectiva, redirecciona o envía por proxy el tráfico al clúster de base de datos correcto.
- Dentro, el tráfico del clúster se reenvía a la base de datos adecuada.

## <a name="connection-policy"></a>Directiva de conexión

Los servidores de SQL Database y Azure Synapse admiten las tres opciones siguientes para la configuración de la Directiva de conexión del servidor:

- **Redirigir (recomendado):** Los clientes establecen conexiones directamente con el nodo que hospeda la base de datos, con lo que se reduce la latencia y se mejora el rendimiento. Para que las conexiones usen este modo, los clientes deben hacer lo siguiente:
  - Permitir la comunicación saliente entre el cliente y todas las direcciones IP de Azure SQL de la región en los puertos en el intervalo de 11000 a 11999. Usar las etiquetas de servicio de SQL para facilitar la administración.  
  - Permitir la comunicación saliente entre el cliente y las direcciones IP de la puerta de enlace de Azure SQL Database en el puerto 1433.

- **Proxy:** En este modo, todas las conexiones se realizan mediante proxy a través de las puertas de enlace de Azure SQL Database, lo que provoca una mayor latencia y un rendimiento inferior. Para que las conexiones usen este modo, los clientes deben permitir la comunicación saliente entre el cliente y las direcciones IP de la puerta de enlace de Azure SQL Database en el puerto 1433.

- **Valor predeterminado:** esta es la directiva de conexión en vigor en todos los servidores después de su creación, a menos que se modifique explícitamente cambiándola a `Proxy` o `Redirect`. La directiva predeterminada es `Redirect` para todas las conexiones de cliente que se originen en Azure (por ejemplo, desde una máquina virtual de Azure) y `Proxy` para todas las conexiones de cliente que se originen fuera (por ejemplo, las conexiones desde la estación de trabajo local).

Es muy recomendable utilizar la directiva de conexión `Redirect` frente a `Proxy` para obtener la menor latencia y el mayor rendimiento. Pero necesitará cumplir los requisitos adicionales para permitir el tráfico de red tal y como se ha descrito anteriormente. Si el cliente es una máquina virtual de Azure, puede hacerlo mediante grupos de seguridad de red (NSG) con [etiquetas de servicio](../../virtual-network/security-overview.md#service-tags). Si el cliente se conecta desde una estación de trabajo local, puede que necesite trabajar con el administrador de red para permitir el tráfico de red a través del firewall corporativo.

## <a name="connectivity-from-within-azure"></a>Conectividad desde dentro de Azure

Si va a conectarse desde dentro de Azure, las conexiones tienen la directiva de conexión predeterminada `Redirect`. Una directiva `Redirect` significa que después de establecer la sesión TCP en Azure SQL Database, la sesión del cliente se redirige al clúster de base de datos correcto con un cambio en la IP virtual de destino de la puerta de enlace de Azure SQL Database a la del clúster. Por lo tanto, todos los paquetes posteriores fluyen directamente al clúster, de tal forma que omiten la puerta de enlace de Azure SQL Database. En el siguiente diagrama, se ilustra este flujo de tráfico.

![Descripción general de la arquitectura](./media/connectivity-architecture/connectivity-azure.png)

## <a name="connectivity-from-outside-of-azure"></a>Conectividad desde fuera de Azure

Si va a conectarse desde fuera de Azure, las conexiones tienen la directiva de conexión predeterminada `Proxy`. La directiva `Proxy` establece que la sesión TCP se realice a través de la puerta de enlace de Azure SQL Database y que todos los paquetes posteriores fluyan a través de la puerta de enlace. En el siguiente diagrama, se ilustra este flujo de tráfico.

![Descripción general de la arquitectura](./media/connectivity-architecture/connectivity-onprem.png)

> [!IMPORTANT]
> Además, abra los puertos TCP 1434 y 14000-14999 para habilitar la [conexión con DAC](https://docs.microsoft.com/sql/database-engine/configure-windows/diagnostic-connection-for-database-administrators?view=sql-server-2017#connecting-with-dac)

## <a name="gateway-ip-addresses"></a>Direcciones IP de puerta de enlace

En la tabla siguiente se muestran las direcciones IP de las puertas de enlace por región. Para conectarse a SQL Database o Azure Synapse, debe permitir el tráfico de red desde y hacia **todas** las puertas de enlace de la región.

En el siguiente artículo se describen los detalles de cómo se migrará el tráfico a las nuevas puertas de enlace de regiones específicas: [Migración de tráfico de Azure SQL Database a puertas de enlace más recientes](gateway-migration.md)

| Nombre de la región          | Direcciones IP de puerta de enlace |
| --- | --- |
| Centro de Australia    | 20.36.105.0 |
| Centro de Australia 2   | 20.36.113.0 |
| Este de Australia       | 13.75.149.87, 40.79.161.1, 13.70.112.9 |
| Sudeste de Australia | 191.239.192.109, 13.73.109.251, 13.77.48.10 |
| Sur de Brasil         | 104.41.11.5, 191.233.200.14 |
| Centro de Canadá       | 40.85.224.249, 52.246.152.0, 20.38.144.1 |
| Este de Canadá          | 40.86.226.166, 52.242.30.154 |
| Centro de EE. UU.           | 13.67.215.62, 52.182.137.15, 23.99.160.139, 104.208.16.96, 104.208.21.1 |
| Este de China           | 139.219.130.35     |
| Este de China 2         | 40.73.82.1         |
| Norte de China          | 139.219.15.17      |
| Norte de China 2        | 40.73.50.0         |
| Este de Asia            | 191.234.2.139, 52.175.33.150, 13.75.32.4 |
| Este de EE. UU.              | 40.121.158.30, 40.79.153.12, 191.238.6.43, 40.78.225.32 |
| Este de EE. UU. 2            | 40.79.84.180, 52.177.185.181, 52.167.104.0,  191.239.224.107, 104.208.150.3 |
| Centro de Francia       | 40.79.137.0, 40.79.129.1 |
| Centro de Alemania      | 51.4.144.100       |
| Nordeste de Alemania   | 51.5.144.179       |
| India central        | 104.211.96.159     |
| Sur de India          | 104.211.224.146    |
| India occidental           | 104.211.160.80     |
| Japón Oriental           | 13.78.61.196, 40.79.184.8, 13.78.106.224, 191.237.240.43, 40.79.192.5 |
| Japón Occidental           | 104.214.148.156, 40.74.100.192, 191.238.68.11, 40.74.97.10 |
| Centro de Corea del Sur        | 52.231.32.42       |
| Corea del Sur          | 52.231.200.86      |
| Centro-Norte de EE. UU     | 23.96.178.199, 23.98.55.75, 52.162.104.33 |
| Norte de Europa         | 40.113.93.91, 191.235.193.75, 52.138.224.1, 13.74.104.113 |
| Este de Noruega          | 51.120.96.0        |
| Oeste de Noruega          | 51.120.216.0       |
| Norte de Sudáfrica   | 102.133.152.0, 102.133.120.2       |
| Oeste de Sudáfrica    | 102.133.24.0       |
| Centro-sur de EE. UU.     | 13.66.62.124, 23.98.162.75, 104.214.16.32, 20.45.121.1, 20.49.88.1   |
| Sudeste de Asia      | 104.43.15.0, 23.100.117.95, 40.78.232.3   |
| Norte de Suiza    | 51.107.56.0, 51.107.57.0 |
| Oeste de Suiza     | 51.107.152.0, 51.107.153.0 |
| Centro de Emiratos Árabes Unidos          | 20.37.72.64        |
| Norte de Emiratos Árabes Unidos            | 65.52.248.0        |
| Sur de Reino Unido             | 51.140.184.11, 51.105.64.0 |
| Oeste de Reino Unido              | 51.141.8.11        |
| Centro-Oeste de EE. UU.      | 13.78.145.25, 13.78.248.43        |
| Oeste de Europa          | 40.68.37.158, 191.237.232.75, 104.40.168.105, 52.236.184.163  |
| Oeste de EE. UU.              | 104.42.238.205, 23.99.34.75, 13.86.216.196   |
| Oeste de EE. UU. 2            | 13.66.226.202, 40.78.240.8, 40.78.248.10  |
|                      |                    |

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo cambiar la directiva de conexión de Azure SQL Database para un servidor, vea [conn-policy](https://docs.microsoft.com/cli/azure/sql/server/conn-policy).
- Para obtener información sobre el comportamiento de conexión de Azure SQL Database para clientes que usan ADO.NET 4.5 o una versión posterior, vea [Puertos más allá de 1433 para ADO.NET 4.5](adonet-v12-develop-direct-route-ports.md).
- Para obtener información general sobre el desarrollo de aplicaciones, vea [Introducción al desarrollo de aplicaciones en SQL Database](develop-overview.md).
