---
title: Parametrización de servicios vinculados en Azure Data Factory
description: Aprenda a parametrizar los servicios vinculados en Azure Data Factory y a pasar valores dinámicos en tiempo de ejecución.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/21/2020
author: djpmsft
ms.author: daperlov
manager: anandsub
ms.openlocfilehash: b3aadab1b4af80f98c57a279b69606a02846e996
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88716850"
---
# <a name="parameterize-linked-services-in-azure-data-factory"></a>Parametrización de servicios vinculados en Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Ahora puede parametrizar un servicio vinculado y pasar valores dinámicos en tiempo de ejecución. Por ejemplo, si quiere conectarse a bases de datos diferentes en el mismo servidor lógico de SQL, ahora puede parametrizar el nombre de la base de datos en la definición del servicio vinculado. Esto evita tener que crear un servicio vinculado para cada base de datos del servidor lógico de SQL. Por ejemplo, puede parametrizar otras propiedades en la definición de servicio vinculado también; por ejemplo, *Nombre de usuario.*

Puede usar la interfaz de usuario de Data Factory en Azure Portal o una interfaz de programación para parametrizar los servicios vinculados.

> [!TIP]
> Se recomienda no parametrizar las contraseñas ni los secretos. Almacene en cambio todas las cadenas de conexión en Azure Key Vault y parametrice el *nombre del secreto*.

Si desea una demostración y una introducción de siete minutos de esta característica, vea el siguiente vídeo:

> [!VIDEO https://channel9.msdn.com/shows/azure-friday/Parameterize-connections-to-your-data-stores-in-Azure-Data-Factory/player]

## <a name="supported-data-stores"></a>Almacenes de datos compatibles

En este momento, la parametrización de los servicios vinculados se admite en la interfaz de usuario de Data Factory para los siguientes almacenes de datos. Para todos los demás almacenes de datos, puede parametrizar el servicio vinculado si selecciona el icono **Código** en la pestaña **Conexiones** y usa el editor de JSON.

- Amazon Redshift
- Azure Cosmos DB (API de SQL)
- Azure Database for MySQL
- Azure SQL Database
- Azure Synapse Analytics (anteriormente SQL DW)
- MySQL
- Oracle
- SQL Server
- HTTP genérico
- REST genérico

## <a name="data-factory-ui"></a>Interfaz de usuario de Data Factory

![Incorporación de contenido dinámico a la definición de servicio vinculado](media/parameterize-linked-services/parameterize-linked-services-image1.png)

![Crear un nuevo parámetro](media/parameterize-linked-services/parameterize-linked-services-image2.png)

## <a name="json"></a>JSON

```json
{
    "name": "AzureSqlDatabase",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Server=tcp:myserver.database.windows.net,1433;Database=@{linkedService().DBName};User ID=user;Password=fake;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        },
        "connectVia": null,
        "parameters": {
            "DBName": {
                "type": "String"
            }
        }
    }
}
```
