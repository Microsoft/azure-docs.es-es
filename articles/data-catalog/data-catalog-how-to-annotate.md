---
title: Anotación de orígenes de datos en Azure Data Catalog
description: Artículo de procedimientos que señala cómo anotar recursos de datos en Azure Data Catalog, incluidos expertos, etiquetas, descripciones y nombres descriptivos.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 61a91ca8a51886c28beee77853d1ae67911e00e7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081243"
---
# <a name="how-to-annotate-data-sources-in-azure-data-catalog"></a>Anotación de orígenes de datos en Azure Data Catalog

## <a name="introduction"></a>Introducción

**Microsoft Azure Data Catalog** es un servicio en la nube totalmente administrado que actúa como sistema de registro y de detección de orígenes de datos empresariales. En otras palabras, Data Catalog ayuda a las personas a detectar, comprender y usar orígenes de datos, así como ayudar a las organizaciones a obtener un mayor valor de sus datos. Cuando un origen de datos se registra en Data Catalog, el servicio copia e indexa sus metadatos, pero eso no es todo. Data Catalog permite que los usuarios faciliten sus propios metadatos descriptivos, como descripciones y etiquetas, para complementar los metadatos extraídos del origen de datos y conseguir que más personas lo comprendan.

## <a name="annotation-and-crowdsourcing"></a>Anotación y micromecenazgo
Todo el mundo tiene una opinión. Y eso es algo bueno.
Data Catalog reconoce que distintos usuarios tienen diferentes perspectivas sobre los orígenes de datos empresariales y que cada una de estas perspectivas puede ser valiosa. Considere el caso siguiente:

* El administrador del sistema conoce el contrato de nivel de servicio de los servidores o servicios que hospedan el origen de datos.
* El administrador de base de datos conoce la programación de copia de seguridad de cada base de datos y las ventanas de procesamiento de ETL permitidas.
* El propietario del sistema conoce el proceso para que los usuarios soliciten acceso al origen de datos.
* El administrador de datos conoce cómo se asignan los recursos y atributos del origen de datos al modelo de datos empresarial.
* El analista conoce cómo se usan los datos en el contexto de los procesos empresariales con los que trabaja.

Cada una de estas perspectivas es importante, y Data Catalog aplica un enfoque de micromecenazgo a los metadatos que permite que cada una se capture y utilice para ofrecer una visión completa de los orígenes de datos registrados. Mediante el portal de Data Catalog, cada usuario puede agregar y modificar sus propias anotaciones, a la vez que puede ver las anotaciones que otros usuarios han proporcionado.

## <a name="different-types-of-annotations"></a>Distintos tipos de anotaciones
La búsqueda de Data Catalog admite los siguientes tipos de anotaciones:

| Anotación | Notas |
| --- | --- |
| Nombre descriptivo |Se pueden proporcionar nombres descriptivos en el nivel de recursos de datos para que dichos recursos de datos se entiendan con más facilidad. Los nombres descriptivos son más útiles cuando el nombre del objeto subyacente es críptico, está abreviado o no es significativo para los usuarios. |
| Descripción |Las descripciones se pueden proporcionar en los niveles de atributos y recursos de datos o de columnas. Las descripciones son anotaciones de texto breve de forma libre que describen la perspectiva del usuario sobre el recurso de datos o su uso. |
| Etiquetas (etiquetas de usuario) |Las etiquetas se pueden proporcionar en los niveles de atributos y recursos de datos o de columnas. Los usuarios definen las etiquetas que se pueden usar para clasificar los atributos o recursos de datos. |
| Etiquetas (etiquetas de glosario) |Las etiquetas se pueden proporcionar en los niveles de atributos y recursos de datos o de columnas. Las etiquetas del glosario son términos de glosario definidos centralmente que pueden utilizarse para clasificar los recursos de datos o atributos mediante una taxonomía empresarial común. Para más información, vea [Configuración del glosario empresarial para el etiquetado regulado](data-catalog-how-to-business-glossary.md). |
| Expertos |Los expertos se pueden facilitar en el nivel de recursos de datos. Los expertos identifican usuarios o grupos con perspectivas expertas sobre los datos, y pueden actuar como puntos de contacto para los usuarios que detecten los orígenes de datos registrados y tengan preguntas que no estén respondidas en las anotaciones existentes. |
| Solicitar acceso |La información de solicitud de acceso se puede proporcionar en el nivel de recursos de datos. Esta información es para los usuarios que detecten un origen de datos al que aún no tienen permisos para tener acceso. Los usuarios pueden escribir la dirección de correo electrónico del usuario o grupo que concede el acceso, la dirección URL del proceso o la herramienta a los que los usuarios necesitan tener acceso, o pueden especificar el propio proceso como texto. |
| Documentación |La documentación se puede facilitar en el nivel de recursos de datos. La documentación sobre recursos es información de texto enriquecido que puede incluir vínculos e imágenes y proporcionar información que no es posible transmitir a través de descripciones y etiquetas. |

## <a name="annotating-multiple-assets"></a>Anotación de varios recursos
Cuando se seleccionan varios recursos de datos en el portal de Data Catalog, los usuarios pueden anotar todos los recursos seleccionados en una sola operación. Las anotaciones se aplicarán a todos los recursos seleccionados, lo que facilita seleccionar y proporcionar una descripción y unos conjuntos de etiquetas y expertos coherentes para los recursos de datos relacionados.

> [!NOTE]
> También se pueden proporcionar etiquetas y expertos al registrar recursos de datos mediante la herramienta de registro de orígenes de datos de Data Catalog.
>
>

Cuando se seleccionan varias tablas y vistas, solo se mostrarán las columnas que tengan en común todos los recursos de datos seleccionados en el portal de Data Catalog. Esto permite que los usuarios proporcionen etiquetas y descripciones para todas las columnas con el mismo nombre de todos los recursos seleccionados.

## <a name="annotations-and-discovery"></a>Anotaciones y detección
Al igual que los metadatos extraídos del origen de datos durante el registro se agregan al índice de búsqueda de Data Catalog, los metadatos proporcionados por los usuarios también se indexan. Esto significa que las anotaciones no solo facilitan que los usuarios entiendan los datos que detectan, sino que también resultará más sencillo para los usuarios la detección de los recursos de datos anotados mediante la búsqueda con los términos que tengan sentido para ellos.

## <a name="summary"></a>Resumen
Al registrar un origen de datos con Data Catalog, se consigue que esos datos sean detectables mediante la copia de los metadatos descriptivos y estructurales del origen de datos en el servicio Catálogo. Cuando se ha registrado un origen de datos, los usuarios pueden proporcionar anotaciones para facilitar la detección y comprensión dentro del portal de Data Catalog.

## <a name="see-also"></a>Consulte también
* [Introducción a Azure Data Catalog](data-catalog-get-started.md) para obtener información paso a paso sobre cómo anotar orígenes de datos.
