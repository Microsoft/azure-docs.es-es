---
title: Uso de orígenes de datos de perfiles de datos en Azure Data Catalog
description: Este artículo de procedimientos destaca cómo incluir perfiles de datos de nivel de tabla y de columna al registrar orígenes de datos en Azure Data Catalog y cómo utilizar perfiles de datos para entender los orígenes de datos.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 0d0f3a45842638915947f5df04e06e2c81a7f5f1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081192"
---
# <a name="how-to-data-profile-data-sources-in-azure-data-catalog"></a>Creación de perfiles de datos de orígenes de datos en Azure Data Catalog

## <a name="introduction"></a>Introducción

**Microsoft Azure Data Catalog** es un servicio en la nube totalmente administrado que actúa como sistema de registro y de detección de orígenes de datos empresariales. En otras palabras, **Azure Data Catalog** consiste en ayudar a las personas a detectar, comprender y usar orígenes de datos, y en ayudar a las organizaciones a obtener más valor de sus datos. Cuando un origen de datos se registra en **Azure Data Catalog**, el servicio copia e indexa sus metadatos, pero eso no es todo.

La característica de **perfiles de datos** de **Azure Data Catalog** examina los datos de orígenes de datos admitidos en el catálogo y recopila estadísticas e información sobre esos datos. Es fácil incluir un perfil de sus recursos de datos. Al registrar un recurso de datos, elija **Incluir perfil de datos** en la herramienta de registro de orígenes de datos.

## <a name="what-is-data-profiling"></a>¿Qué es la generación de perfiles de datos?

La generación de perfiles de datos examina los datos del origen de datos que se registra y recopila estadísticas e información sobre esos datos. Durante la detección del origen de datos, estas estadísticas pueden ayudar a los usuarios a determinar la idoneidad de los datos para resolver sus problemas empresariales.

<!-- In [How to discover data sources](data-catalog-how-to-discover.md), you learn about **Azure Data Catalog's** extensive search capabilities including searching for data assets that have a profile. See [How to include a data profile when registering a data source](#howto). -->

Los siguientes orígenes de datos admiten la generación de perfiles de datos:

* Vistas y tablas de SQL Server (incluidos SQL Data Warehouse y Azure SQL Database)
* Vistas y tablas de Oracle
* Vistas y tablas de Teradata
* Tablas de Hive

Incluir los perfiles de datos al registrar lo recursos de datos ayuda a los usuarios a responder preguntas acerca de los orígenes de datos, incluidas:

* ¿Puede utilizarse para solucionar mi problema empresarial?
* ¿Los datos se ajustan a estándares o patrones específicos?
* ¿Cuáles son algunas de las anomalías del origen de datos?
* ¿Cuáles son los posibles retos de integración de estos datos en mi aplicación?

> [!NOTE]
> También puede agregar documentación a un recurso para describir cómo se pueden integrar los datos en una aplicación. Consulte [Documentación de los orígenes de datos](data-catalog-how-to-documentation.md).
>

<a name="howto"></a>

## <a name="how-to-include-a-data-profile-when-registering-a-data-source"></a>Cómo incluir un perfil de datos al registrar un origen de datos

Es fácil incluir un perfil de sus origen de datos. Al registrar un origen de datos, en el panel **Objetos que se registrarán** de la herramienta de registro de orígenes de datos, elija **Incluir perfil de datos**.

![Casilla Incluir perfil de datos](media/data-catalog-data-profile/data-catalog-register-profile.png)

Para más información sobre cómo registrar orígenes de datos, vea [Registro de orígenes de datos](data-catalog-how-to-register.md) e [Introducción a Azure Data Catalog](data-catalog-get-started.md).

## <a name="filtering-on-data-assets-that-include-data-profiles"></a>Filtrado de recursos de datos que incluyen perfiles de datos

Para detectar los recursos de datos que incluyen un perfil de datos, puede incluir `has:tableDataProfiles` o `has:columnsDataProfiles` como uno de los términos de búsqueda.

> [!NOTE]
> Al seleccionar **Incluir perfil de datos** en la herramienta de registro de orígenes de datos, se incluye la información de perfil de nivel de columna y tabla. Sin embargo, la API de Data Catalog permite que los recursos de datos se registren con un único conjunto de información de perfil.
>

## <a name="viewing-data-profile-information"></a>Visualización de la información del perfil de datos

Una vez que encuentre un origen de datos adecuado con un perfil, puede ver los detalles del perfil de datos. Para ver el perfil de datos, seleccione un recurso de datos y elija **Perfil de datos** en la ventana del portal de Catálogo de datos.

![Pestaña Perfil de datos](media/data-catalog-data-profile/data-catalog-view.png)

Un perfil de datos del **Azure Data Catalog** muestra la información del perfil de tabla y columna, incluido lo siguiente:

### <a name="object-data-profile"></a>Perfil de datos de objeto

* Número de filas
* Tamaño de la tabla
* Cuándo se actualizó por última vez el objeto

### <a name="column-data-profile"></a>Perfil de datos de columna

* Tipo de datos de columna
* Número de valores distintivos
* Número de filas con valores NULL
* Mínimo, máximo, promedio y desviación estándar para los valores de las columnas

## <a name="summary"></a>Resumen

La generación de perfiles de datos proporciona estadísticas e información sobre los recursos de datos registrados para ayudar a los usuarios a determinar la idoneidad de los datos para resolver problemas empresariales. Junto con la anotación y documentación de los orígenes de datos, los perfiles de datos pueden dar a los usuarios una comprensión más profunda de los datos.

## <a name="see-also"></a>Consulte también

* [Registro de orígenes de datos](data-catalog-how-to-register.md)
* [Introducción a Azure Data Catalog](data-catalog-get-started.md)
