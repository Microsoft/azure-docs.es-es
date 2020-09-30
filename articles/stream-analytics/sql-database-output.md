---
title: Salida de Azure SQL Database desde Azure Stream Analytics
description: En este artículo se describe Azure SQL Database como salida de Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 4310bd94edd5ebe14eab40b6d19e2bacbdd1b03c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906224"
---
# <a name="azure-sql-database-output-from-azure-stream-analytics"></a>Salida de Azure SQL Database desde Azure Stream Analytics

Puede usar [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) como salida de datos que son relacionales por naturaleza o de aplicaciones que dependen del contenido hospedado en una base de datos relacional. Los trabajos de Azure Stream Analytics se escriben en una tabla existente en SQL Database. El esquema de tabla debe coincidir exactamente con los campos y los tipos en la salida del trabajo. También puede especificar [Azure Synapse Analytics](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) como salida mediante la opción de salida de SQL Database. Para obtener más información sobre cómo mejorar el rendimiento de escritura, consulte el artículo [Stream Analytics con Azure SQL Database como salida](stream-analytics-sql-output-perf.md).

También puede usar [Instancia administrada de Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance) como salida. Debe [configurar un punto de conexión público en Instancia administrada de SQL](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-public-endpoint-configure) y, luego, configurar manualmente las siguientes opciones en Azure Stream Analytics. También es posible configurar manualmente los valores siguientes para una máquina virtual de Azure que ejecute SQL Server con una base de datos adjunta.

## <a name="output-configuration"></a>Configuración de salida

En la siguiente tabla se enumeran los nombres de propiedad y su descripción para crear una salida de SQL Database.

| Nombre de propiedad | Descripción |
| --- | --- |
| Alias de salida |Un nombre descriptivo usado en las consultas para dirigir la salida de la consulta a esta base de datos. |
| Base de datos | Nombre de la base de datos adonde envía la salida. |
| Nombre de servidor | El nombre de servidor SQL lógico o el nombre de instancia administrada. Para Instancia administrada de SQL, es necesario especificar el puerto 3342. Por ejemplo, *sampleserver.public.database.windows.net,3342*. |
| Nombre de usuario | Nombre de usuario que tiene acceso de escritura a la base de datos. Stream Analytics admite solo la autenticación de SQL. |
| Contraseña | La contraseña para conectarse a la base de datos. |
| Tabla | El nombre de la tabla donde se escribe la salida. El nombre de la tabla distingue mayúsculas de minúsculas. El esquema de esta tabla debe coincidir exactamente con el número y tipo de los campos que genera la salida del trabajo. |
|Heredación del esquema de partición| Opción para heredar el esquema de partición del paso de consulta anterior a fin de habilitar la topología completamente paralela con múltiples escritores en la tabla. Para obtener más información, vea [Salida de Azure Stream Analytics a Azure SQL Database](stream-analytics-sql-output-perf.md).|
|Número máximo de lotes| Número máximo de registros recomendado para enviarse con cada transacción de inserción masiva.|

Hay dos adaptadores que permiten la salida de Azure Stream Analytics a Azure Synapse Analytics (anteriormente SQL Data Warehouse): SQL Database y Azure Synapse. Se recomienda elegir el adaptador de Azure Synapse Analytics en lugar del adaptador de SQL Database si alguna de las condiciones siguientes es verdadera:

* **Rendimiento**: Si el rendimiento esperado ahora o en el futuro es superior a 10 MB/s, use la opción de salida de Azure Synapse para mejorar el rendimiento.

* **Particiones de entrada**: Si tiene ocho o más particiones de entrada, use la opción de salida de Azure Synapse para mejorar la escalabilidad horizontal.

## <a name="partitioning"></a>Creación de particiones

La creación de particiones debe estar habilitada y se basa en la cláusula PARTITION BY de la consulta. Cuando se habilita la opción para heredar particiones, se siguen las particiones de entrada para [las consultas que se pueden paralelizar totalmente](stream-analytics-scale-jobs.md). Para obtener más información sobre cómo conseguir un mejor rendimiento de escritura al cargar datos en Azure SQL Database, consulte [Salida de Azure Stream Analytics a Azure SQL Database](stream-analytics-sql-output-perf.md).

## <a name="output-batch-size"></a>Tamaño de lote de salida

Puede configurar el tamaño máximo del mensaje mediante **Número máximo de lotes**. El valor máximo predeterminado es 10 000 y el mínimo predeterminado es 100 filas por inserción masiva única. Para más información, consulte los [límites de SQL de Azure](../sql-database/sql-database-resource-limits.md). Inicialmente, todos los lotes se insertan de forma masiva con el número máximo de lotes. El lote se divide por la mitad (hasta alcanzar el número mínimo de lotes) según los errores con posibilidad de reintento de SQL.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-visual-studio-code.md)
