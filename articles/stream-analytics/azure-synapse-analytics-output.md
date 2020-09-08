---
title: Salida de Azure Synapse Analytics desde Azure Stream Analytics
description: En este artículo se describe Azure Synapse Analytics como salida para Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 5d281bf33542354e2904805deed2f1deaf2725a2
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88875534"
---
# <a name="azure-synapse-analytics-output-from-azure-stream-analytics-preview"></a>Salida de Azure Synapse Analytics desde Azure Stream Analytics (versión preliminar)

[Azure Synapse Analytics](https://azure.microsoft.com/services/synapse-analytics) (anteriormente SQL Data Warehouse) es un servicio de análisis ilimitado que reúne el almacenamiento de datos empresariales y el análisis de macrodatos. 

La salida de los trabajos de Azure Stream Analytics se puede generar en una tabla de grupos de SQL en Azure Synapse Analytics y pueden procesar velocidades de rendimiento de hasta 200 MB/s. Esto admite los análisis en tiempo real más exigentes y las necesidades de procesamiento de datos de ruta de acceso activa para cargas de trabajo como generación de informes y creación de paneles.  

La tabla de grupo de SQL debe existir antes de poder agregarla como salida al trabajo de Stream Analytics. El esquema de tabla debe coincidir con los campos y los tipos en la salida del trabajo. 

Para usar Azure Synapse como salida, debe asegurarse de que tiene la cuenta de almacenamiento configurada. Vaya a Configuración de cuenta de almacenamiento para configurar la cuenta de almacenamiento. Solo se permiten los tipos de cuenta de almacenamiento que admiten tablas: Uso general V2 y uso general V1. Seleccione solo el nivel estándar. No se admite el nivel Premium.

## <a name="output-configuration"></a>Configuración de salida

En la siguiente tabla se enumeran los nombres de propiedad y su descripción para crear una salida de Azure Synapse Analytics.

|Nombre de propiedad|Descripción|
|-|-|
|Alias de salida |Un nombre descriptivo usado en las consultas para dirigir la salida de la consulta a esta base de datos. |
|Base de datos |Nombre del grupo de SQL adonde envía la salida. |
|Nombre de servidor |Nombre del servidor de Azure Synapse.  |
|Nombre de usuario |Nombre de usuario que tiene acceso de escritura a la base de datos. Stream Analytics admite solo la autenticación de SQL. |
|Contraseña |La contraseña para conectarse a la base de datos. |
|Tabla  | El nombre de la tabla donde se escribe la salida. El nombre de la tabla distingue mayúsculas de minúsculas. El esquema de esta tabla debe coincidir exactamente con el número y tipo de los campos que genera la salida del trabajo.|

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-vs-code.md)