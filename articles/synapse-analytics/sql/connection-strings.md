---
title: Cadenas de conexión para Synapse SQL (versión preliminar)
description: Cadenas de conexión para Synapse SQL (versión preliminar)
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick
ms.custom: devx-track-csharp
ms.openlocfilehash: 42a1110b089d5edf9793bf2bb2c699a717a4b1a4
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004976"
---
# <a name="connection-strings-for-synapse-sql-preview"></a>Cadenas de conexión para Synapse SQL (versión preliminar)

Puede conectarse a Synapse SQL con varios protocolos de aplicación diferentes, como [ADO.NET](https://msdn.microsoft.com/library/e80y5yhx(v=vs.110).aspx), [ODBC](https://msdn.microsoft.com/library/jj730314.aspx), [PHP](https://msdn.microsoft.com/library/cc296172.aspx?f=255&MSPPError=-2147217396) y [JDBC](https://msdn.microsoft.com/library/mt484311(v=sql.110).aspx). A continuación se muestran algunos ejemplos de cadenas de conexión para cada protocolo. 

También puede usar el Portal de Azure para crear la cadena de conexión.  Para generar la cadena de conexión mediante Azure Portal, vaya a la hoja de la base de datos en *Aspectos básicos* y haga clic en *Mostrar cadenas de conexión de base de datos*.

## <a name="sample-adonet-connection-string"></a>Cadena de conexión ADO.NET de ejemplo

```csharp
Server=tcp:{your_server}.sql.azuresynapse.net,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

## <a name="sample-odbc-connection-string"></a>Cadena de conexión ODBC de ejemplo

```csharp
Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.sql.azuresynapse.net,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

## <a name="sample-php-connection-string"></a>Cadena de conexión PHP de ejemplo

```PHP
Server: {your_server}.sql.azuresynapse.net,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( \"sqlsrv:server = tcp:{your_server}.sql.azuresynapse.net,1433; Database = {your_database}\", \"{your_user_name}\", \"{your_password_here}\");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( \"Error connecting to SQL Server.\" );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array(\"UID\" => \"{your_user_name}\", \"pwd\" => \"{your_password_here}\", \"Database\" => \"{your_database}\", \"LoginTimeout\" => 30, \"Encrypt\" => 1, \"TrustServerCertificate\" => 0);\r\n$serverName = \"tcp:{your_server}.sql.azuresynapse.net,1433\";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);
```

## <a name="sample-jdbc-connection-string"></a>Cadena de conexión JDBC de ejemplo

```Java
jdbc:sqlserver://yourserver.sql.azuresynapse.net:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.sql.azuresynapse.net;loginTimeout=30;
```

> [!NOTE]
> Considere la posibilidad de establecer el tiempo de espera de conexión en 300 segundos para permitir que la conexión se conserve durante breves períodos de falta de disponibilidad.

## <a name="recommendations"></a>Recomendaciones

Para ejecutar consultas de **SQL a petición**, se recomienda usar estas dos herramientas: [Azure Data Studio](get-started-azure-data-studio.md) y Azure Synapse Studio.

## <a name="next-steps"></a>Pasos siguientes

Para empezar a realizar consultas en los análisis con Visual Studio y otras aplicaciones, consulte [Consulta con Visual Studio](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
