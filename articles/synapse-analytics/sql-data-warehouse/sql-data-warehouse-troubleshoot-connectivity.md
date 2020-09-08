---
title: Solución de problemas de conectividad
description: Solución de problemas de conectividad en el grupo de SQL de Synapse.
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/27/2019
ms.author: anjangsh
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse, devx-track-csharp
ms.openlocfilehash: 2edb3060437080e528d41d4af5f4affd4fbf3316
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89010195"
---
# <a name="troubleshooting-connectivity-issues-in-synapse-sql-pool"></a>Solución de problemas de conectividad en el grupo de Synapse SQL

En este artículo se enumeran las técnicas de solución de problemas comunes en torno a la conexión de una base de datos de grupo de SQL.

## <a name="check-service-availability"></a>Comprobación de la disponibilidad del servicio

Compruebe si el servicio está disponible. En Azure Portal, vaya al grupo de SQL al que está intentando conectarse. En el panel de índice izquierdo, haga clic en **Diagnosticar y solucionar problemas**.

![Selección de Resource Health](./media/sql-data-warehouse-troubleshoot-connectivity/diagnostics-link.png)

El estado de su grupo de SQL se mostrará aquí. Si el servicio no se muestra como **disponible**, consulte más pasos.

![El servicio está disponible](./media/sql-data-warehouse-troubleshoot-connectivity/resource-health.png)

Si Resource Health muestra que la instancia del grupo de SQL está en pausa o se está escalando, siga las instrucciones para reanudarla.

![Servicio pausado](./media/sql-data-warehouse-troubleshoot-connectivity/resource-health-pausing.png) Aquí puede encontrar información adicional sobre Resource Health.

## <a name="check-for-paused-or-scaling-operation"></a>Comprobación de la operación de pausa o escalado

Compruebe en el portal si su instancia del grupo de SQL está en pausa o se está escalando.

![Servicio en pausa](./media/sql-data-warehouse-troubleshoot-connectivity/overview-paused.png)

Si ve el servicio está en pausa o escalado, compruebe que no es durante la programación de mantenimiento. En el portal, en la página *Información general* del grupo de SQL, verá la programación de mantenimiento elegida.

![Información general de la programación de mantenimiento](./media/sql-data-warehouse-troubleshoot-connectivity/overview-maintance-schedule.png)

En caso contrario, consulte al Administrador de TI para comprobar que este mantenimiento no es un evento programado. Para reanudar la instancia del grupo de SQL, siga [estos pasos](pause-and-resume-compute-portal.md).

## <a name="check-your-firewall-settings"></a>Comprobación de la configuración de firewall

La base de datos del grupo de SQL se comunica a través del puerto 1433.  Si intenta conectarse desde una red corporativa, es posible que el firewall de la red no permita el tráfico saliente a través del puerto 1433. En ese caso, no podrá conectarse al [servidor lógico](../../azure-sql/database/logical-servers.md) a menos que el departamento de TI abra el puerto 1433. Puede encontrar información adicional sobre las configuraciones de firewall [aquí](../../azure-sql/database/firewall-configure.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#create-and-manage-ip-firewall-rules).

## <a name="check-your-vnetservice-endpoint-settings"></a>Comprobación de la configuración del punto de conexión de servicio o red virtual

Si recibe los errores 40914 y 40615, consulte la [descripción del error y la resolución aquí](../../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#errors-40914-and-40615).

## <a name="check-for-the-latest-drivers"></a>Comprobación de los controladores más recientes

### <a name="software"></a>Software

Asegúrese de usar las herramientas más recientes para conectarse al grupo de SQL:

- SSMS
- Azure Data Studio
- SQL Server Data Tools (Visual Studio)

### <a name="drivers"></a>Controladores

Asegúrese de usar las versiones más recientes del controlador.  El uso de una versión anterior de los controladores podría provocar comportamientos inesperados como que los controladores más antiguos no admitiesen las nuevas características.

- [ODBC](/sql/connect/odbc/download-odbc-driver-for-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [JDBC](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [OLE DB](/sql/connect/oledb/download-oledb-driver-for-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [PHP](/sql/connect/php/download-drivers-php-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

## <a name="check-your-connection-string"></a>Comprobación de la cadena de conexión

Asegúrese de que las cadenas de conexión se han establecido correctamente.  A continuación se muestran algunos ejemplos.  También puede encontrar información adicional sobre [las cadenas de conexión aquí](sql-data-warehouse-connection-strings.md).

Cadena de conexión ADO.NET

```csharp
Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

Cadena de conexión ODBC

```csharp
Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

Cadena de conexión PHP

```PHP
Server: {your_server}.database.windows.net,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( \"sqlsrv:server = tcp:{your_server}.database.windows.net,1433; Database = {your_database}\", \"{your_user_name}\", \"{your_password_here}\");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( \"Error connecting to SQL Server.\" );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array(\"UID\" => \"{your_user_name}\", \"pwd\" => \"{your_password_here}\", \"Database\" => \"{your_database}\", \"LoginTimeout\" => 30, \"Encrypt\" => 1, \"TrustServerCertificate\" => 0);\r\n$serverName = \"tcp:{your_server}.database.windows.net,1433\";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);
```

Cadena de conexión JDBC

```Java
jdbc:sqlserver://yourserver.database.windows.net:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
```

## <a name="intermittent-connection-issues"></a>Problemas de conexión intermitente

Compruebe si experimenta mucha carga en el servidor con un gran número de solicitudes en cola. Es posible que deba escalar verticalmente el grupo de SQL para obtener recursos adicionales.

## <a name="common-error-messages"></a>Mensajes comunes de error

Con los errores 40914 y 40615, consulte la [descripción del error y la resolución aquí](../../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#errors-40914-and-40615).

## <a name="still-having-connectivity-issues"></a>¿Sigue teniendo problemas de conectividad?

Cree una [incidencia de soporte técnico](sql-data-warehouse-get-started-create-support-ticket.md) para que el equipo de ingeniería pueda ayudarle.
