---
title: Documentación de orígenes de datos en Azure Data Catalog
description: En este artículo de procedimientos se resalta cómo documentar recursos de datos en Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 3674c316e34b1e54ed8282ac7d2c228a2b774c06
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081226"
---
# <a name="how-to-document-data-sources-in-azure-data-catalog"></a>Documentación de orígenes de datos en Azure Data Catalog

## <a name="introduction"></a>Introducción
**Microsoft Azure Data Catalog** es un servicio en la nube totalmente administrado que actúa como sistema de registro y de detección de orígenes de datos empresariales. En otras palabras, el **Azure Data Catalog** ayuda a las personas a detectar, *comprender*y usar orígenes de datos, así como ayudar a las organizaciones a obtener un mayor valor de sus datos.

Cuando un origen de datos se registra en **Azure Data Catalog**, el servicio copia e indexa sus metadatos, pero eso no es todo. **Azure Data Catalog** también permite a los usuarios proporcionar su propia documentación completa, que puede describir el uso y los escenarios comunes para el origen de datos.

En [Anotación de orígenes de datos](data-catalog-how-to-annotate.md), sabrá que los expertos que conocen el origen de datos pueden anotarlo con etiquetas y una descripción. El portal del **Azure Data Catalog** incluye un editor de texto enriquecido para que los usuarios puedan documentar completamente contenedores y recursos de datos. El editor incluye el formato de párrafo, como los títulos, formato de texto, listas con viñetas, listas numeradas y tablas.

Las etiquetas y descripciones son excelentes para anotaciones simples. Sin embargo, para ayudar a los consumidores de datos a comprender mejor el uso de un origen de datos y los escenarios empresariales para un origen de datos, un experto puede proporcionar documentación completa y detallada. Es fácil documentar un origen de datos. Simplemente, seleccione un recurso de datos o un contenedor y elija **Documentación**.

![Pestaña de documentación en una instancia de Data Catalog](media/data-catalog-documentation/data-catalog-documentation.png)

## <a name="documenting-data-assets"></a>Documentación de los recursos de datos
La ventaja del **Azure Data Catalog** es que la documentación le permite usar Data Catalog como un repositorio de contenido para crear una descripción completa de los recursos de datos. Puede explorar el contenido detallado que describe los contenedores y las tablas. Si ya tiene contenido en otro repositorio de contenido, como SharePoint o un recurso compartido de archivos, puede agregar los vínculos de documentación de recursos para hacer referencia a este contenido existente. Esta característica hace que los documentos existentes sean más detectables.

> [!NOTE]
> La documentación no se incluye en el índice de búsqueda.
>

![Pestaña de documentación e hipervínculo a un vínculo web](media/data-catalog-documentation/data-catalog-documentation2.png)

El nivel de documentación puede oscilar desde describir las características y el valor de un contenedor de recursos de datos para una descripción detallada del esquema de tabla dentro de un contenedor. El nivel de documentación proporcionado debe basarse en las necesidades del negocio. En general, hay algunas ventajas y desventajas de documentar los recursos de datos:

* Documentar solo un contenedor: todo el contenido está en un solo lugar, pero puede faltar información necesaria para que los usuarios tomen una decisión informada.
* Documentar solo las tablas: el contenido es específico de ese objeto, pero los usuarios tienen varios lugares para los documentos.
* Documentar contenedores y tablas: el enfoque más completo, pero puede incluir un mayor mantenimiento de los documentos.

## <a name="summary"></a>Resumen
La documentación de orígenes de datos con **Azure Data Catalog** puede crear una descripción de los recursos de datos con el detalle que sea necesario.  Mediante el uso de vínculos, puede vincular al contenido almacenado en un repositorio de contenido existente, lo que reúne los recursos de datos y los documentos existentes. Una vez que los usuarios descubran los recursos de datos adecuados, pueden tener un completo conjunto de documentación.
