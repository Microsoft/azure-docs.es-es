---
title: Configuración de una conexión a un origen de datos mediante una identidad administrada (versión preliminar)
titleSuffix: Azure Cognitive Search
description: Obtenga información sobre cómo configurar una conexión de indexador a un origen de datos mediante una identidad administrada (versión preliminar)
manager: luisca
author: markheff
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/18/2020
ms.openlocfilehash: d303de23a04d183d0ca280c3b3591299d883adf7
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936595"
---
# <a name="set-up-an-indexer-connection-to-a-data-source-using-a-managed-identity-preview"></a>Configuración de una conexión de indexador a un origen de datos mediante una identidad administrada (versión preliminar)

> [!IMPORTANT] 
> La compatibilidad para configurar una conexión a un origen de datos utilizando una identidad administrada se encuentra actualmente en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción.

Un [indexador](search-indexer-overview.md) de Azure Cognitive Search es un rastreador que proporciona una manera de extraer datos del origen de datos en Azure Cognitive Search. Un indexador obtiene una conexión de origen de datos de un objeto de origen de datos creado por usted. El objeto de origen de datos normalmente incluye las credenciales del origen de datos de destino. Por ejemplo, el objeto de origen de datos podría incluir una clave de cuenta de Azure Storage si desea indexar datos de un contenedor de Blob Storage.

En muchos casos, proporcionar credenciales directamente en el objeto de origen de datos no es un problema, pero pueden presentarse algunos desafíos:
* ¿Cómo puedo proteger las credenciales en mi código que crea el objeto de origen de datos?
* Si la contraseña o la clave de la cuenta se ve comprometida y necesito cambiarla, ahora necesito actualizar mis objetos de origen de datos con la nueva clave o contraseña de la cuenta para que mi indexador pueda conectarse de nuevo al origen de datos.

Estos problemas pueden resolverse mediante la configuración de la conexión con una identidad administrada.

## <a name="using-managed-identities"></a>Uso de identidades administradas

Las [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory (Azure AD). Puede usar esta característica en Azure Cognitive Search para crear un objeto de origen de datos con una cadena de conexión que no incluya ninguna credencial. En su lugar, se concederá al servicio de búsqueda acceso al origen de datos mediante el control de acceso basado en rol (RBAC).

Al configurar un origen de datos mediante una identidad administrada, puede cambiar las credenciales del origen de datos y los indexadores seguirán pudiendo conectarse al origen de datos. También puede crear objetos de origen de datos en el código sin tener que incluir una clave de cuenta ni usar Key Vault para recuperar una clave de cuenta.

## <a name="limitations"></a>Limitaciones

Los orígenes de datos siguientes admiten la configuración de una conexión de indexador mediante identidades administradas. 

* [Azure Blob Storage, Azure Data Lake Storage Gen2 (versión preliminar), Azure Table Storage](search-howto-managed-identities-storage.md)
* [Azure Cosmos DB](search-howto-managed-identities-cosmos-db.md)
* [Azure SQL Database](search-howto-managed-identities-sql.md)

Las siguientes características no admiten actualmente el uso de identidades administradas para configurar la conexión:
* Knowledge Store
* Aptitudes personalizadas
 
## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo configurar una conexión de indexador mediante identidades administradas:

* [Azure Blob Storage, Azure Data Lake Storage Gen2 (versión preliminar), Azure Table Storage](search-howto-managed-identities-storage.md)
* [Azure Cosmos DB](search-howto-managed-identities-cosmos-db.md)
* [Azure SQL Database](search-howto-managed-identities-sql.md)