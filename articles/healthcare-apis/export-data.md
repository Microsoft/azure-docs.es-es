---
title: Ejecución de la exportación mediante la invocación del comando $export en Azure API for FHIR
description: En este artículo se indica cómo configurar y usar la exportación sin identificación.
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 8/26/2020
ms.author: matjazl
ms.openlocfilehash: 10d901f73006051e8b1ddd02aeb36b229c6a7761
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269921"
---
# <a name="how-to-export-fhir-data"></a>Exportación de datos de FHIR

Para configurar las opciones de exportación y crear una cuenta de Azure Storage, consulte [esta información](configure-export-data.md).

## <a name="exporting-fhir-resources-using-export-command"></a>Exportación de recursos de FHIR mediante el comando $export

Una vez que hemos configurado Azure API for FHIR para la exportación, podemos usar el comando $export para exportar los datos del servicio a la cuenta de almacenamiento que se ha especificado al configurar la exportación. Para más información sobre cómo invocar el comando $export en el servidor de FHIR, lea la documentación sobre la especificación de dicho comando en [https://hl7.org/Fhir/uv/bulkdata/export/index.html](https://hl7.org/Fhir/uv/bulkdata/export/index.html). 

El comando $export de Azure API for FHIR toma un parámetro opcional _\_container_ que se puede usar para especificar el contenedor de la cuenta de almacenamiento configurada a la que se deben exportar los datos.

`https://<<FHIR service base URL>>/$export?_container=<<container_name>>`

> [!IMPORTANT]
> Tenga en cuenta que actualmente Azure API for FHIR solo admite la exportación en el nivel de sistema, como se define en la especificación de $export en [https://hl7.org/Fhir/uv/bulkdata/export/index.html](https://hl7.org/Fhir/uv/bulkdata/export/index.html). Además, solo se admite actualmente el parámetro de consulta _\_since_.

## <a name="exporting-de-identified-data-preview"></a>Exportación de datos sin identificación (versión preliminar)

También se puede usar el comando $export para exportar datos sin identificación desde el servidor de FHIR. Usa el motor de anonimización de [Herramientas de FHIR para la anonimización](https://github.com/microsoft/FHIR-Tools-for-Anonymization) y toma los detalles de configuración de anonimización en los parámetros de la consulta. Puede crear su propio archivo de configuración de anonimización o usar el [archivo de configuración de ejemplo](https://github.com/microsoft/FHIR-Tools-for-Anonymization#sample-configuration-file-for-hipaa-safe-harbor-method) para el método Safe Harbor de HIPAA como punto de partida. 

 `https://<<FHIR service base URL>>/$export?_container=<<container_name>>&_anonymizationConfig=<<config file name>>&_anonymizationConfigEtag=<<ETag on storage>>`

|Parámetro de consulta            | Ejemplo |Opcionalidad| Descripción|
|---------------------------|---------|-----------|------------|
| _\_anonymizationConfig_   |DemoConfig.json|Obligatorio para la exportación sin identificación |Nombre del archivo de configuración. Consulte el formato del archivo de configuración [aquí](https://github.com/microsoft/FHIR-Tools-for-Anonymization#configuration-file-format). Este archivo debe mantenerse dentro de un contenedor llamado **anonymization** dentro de la misma cuenta de almacenamiento de Azure que está configurada como ubicación de exportación. |
| _\_anonymizationConfigEtag_|"0x8D8494A069489EC"|Opcional para la exportación sin identificación|Este es el valor de Etag del archivo de configuración. Puede obtener el valor de Etag mediante Explorador de Azure Storage desde la propiedad del blob.|

> [!IMPORTANT]
> Tenga en cuenta que tanto la exportación sin formato como la exportación sin identificación escriben en la misma cuenta de almacenamiento de Azure especificada como parte de la configuración de la exportación. Se recomienda que use distintos contenedores que se correspondan con las diferentes configuraciones sin identificación y que administren el acceso de usuarios en el nivel de contenedor.
