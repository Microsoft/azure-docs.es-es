---
title: Copia de datos desde un origen REST mediante Azure Data Factory | Microsoft Docs
description: Obtenga información sobre cómo copiar datos desde un origen REST, en la nube o en un entorno local, a almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 03/28/2019
ms.author: jingwang
ms.openlocfilehash: ee47f464c59bd9deed98671f19cfcc6d2c3c1b39
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2019
ms.locfileid: "60546649"
---
# <a name="copy-data-from-a-rest-endpoint-by-using-azure-data-factory"></a>Copia de datos desde un punto de conexión REST mediante Azure Data Factory

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde un punto de conexión REST. El artículo se basa en [Actividad de copia en Azure Data Factory](copy-activity-overview.md), en el que se ofrece información general acerca de la actividad de copia.

Las diferencias entre este conector REST, el [conector HTTP](connector-http.md) y el [conector de tabla web](connector-web-table.md) son:

- El **conector REST** admite específicamente la copia de datos desde API RESTful; 
- El **conector HTTP** es genérico y puede recuperar datos desde cualquier punto de conexión HTTP, por ejemplo, para descargar archivos. Antes de que esté disponible este conector REST, puede usar el conector HTTP para copiar datos de la API RESTful, lo que se admite, pero es menos funcional en comparación con el conector REST.
- El **conector de tabla web** extrae contenido de la tabla de una página web HTML.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Puede copiar datos desde un origen REST a cualquier almacén de datos receptor compatible. Para obtener una lista de almacenes de datos que la actividad de copia admite como orígenes y receptores, consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).

En concreto, este conector REST genérico admite lo siguiente:

- Recuperar datos desde un punto de conexión REST mediante los métodos **GET** o **POST**.
- Recuperar datos mediante una de las siguientes autenticaciones: **Anónima**, **Básica**, **Entidad de servicio de AAD** e **identidades administradas para recursos de Azure**.
- **[Paginación](#pagination-support)** en las API REST.
- Copiar la respuesta JSON REST [tal cual](#export-json-response-as-is) o analizarla mediante la [asignación de esquema](copy-activity-schema-and-type-mapping.md#schema-mapping). Solo se admite la carga de respuesta en **JSON**.

> [!TIP]
> Para probar una solicitud REST para la recuperación de datos antes de configurar el conector REST en Data Factory, obtenga información sobre la especificación de API para los requisitos del cuerpo y del encabezado. Puede usar herramientas como Postman o un explorador web para validar.

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que puede usar para definir entidades de Data Factory específicas del conector REST.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de REST:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | El **tipo** propiedad debe establecerse en **RestService**. | Sí |
| url | La dirección URL base del servicio REST. | Sí |
| enableServerCertificateValidation | Si se debe validar el certificado SSL del lado servidor al conectarse al punto de conexión. | Sin <br /> (El valor predeterminado es: **true**) |
| authenticationType | El tipo de autenticación usado para conectarse al servicio REST. Los valores que se permiten son: **Anónima**, **Básica**, **AadServicePrincipal** y **ManagedServiceIdentity**. Haga referencia a las siguientes secciones correspondientes para obtener más información sobre propiedades y ejemplos, respectivamente. | Sí |
| connectVia | Instancia de [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un IR autohospedado (si el almacén de datos se encuentra en una red privada). Si no se especifica, esta propiedad se usará Azure Integration Runtime. |Sin  |

### <a name="use-basic-authentication"></a>Uso de la autenticación básica

Establezca la propiedad **authenticationType** en **Basic**. Además de las propiedades genéricas descritas en las secciones anteriores, especifique las siguientes:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| userName | El nombre de usuario para acceder al punto de conexión REST. | Sí |
| password | Contraseña del usuario (valor **userName**). Marque este campo como de tipo **SecureString** para almacenarlo de forma segura en Data Factory. También puede [hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |

**Ejemplo**

```json
{
    "name": "RESTLinkedService",
    "properties": {
        "type": "RestService",
        "typeProperties": {
            "authenticationType": "Basic",
            "url" : "<REST endpoint>",
            "userName": "<user name>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="use-aad-service-principal-authentication"></a>Uso de la autenticación de entidad de servicio de AAD

Establezca la propiedad **authenticationType** en **AadServicePrincipal**. Además de las propiedades genéricas descritas en las secciones anteriores, especifique las siguientes:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| servicePrincipalId | Especifique el identificador de cliente de la aplicación de Azure Active Directory. | Sí |
| servicePrincipalKey | Especifique la clave de la aplicación de Azure Active Directory. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**. | Sí |
| tenant | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarla, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí |
| aadResourceId | Especifique el recurso de AAD para el que solicita autorización, por ejemplo, `https://management.core.windows.net`.| Sí |

**Ejemplo**

```json
{
    "name": "RESTLinkedService",
    "properties": {
        "type": "RestService",
        "typeProperties": {
            "url": "<REST endpoint e.g. https://www.example.com/>",
            "authenticationType": "AadServicePrincipal",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "value": "<service principal key>",
                "type": "SecureString"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "aadResourceId": "<AAD resource URL e.g. https://management.core.windows.net>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identity"></a> Uso de identidades administradas para la autenticación de recursos de Azure

Establezca la propiedad **authenticationType** en **ManagedServiceIdentity**. Además de las propiedades genéricas descritas en las secciones anteriores, especifique las siguientes:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| aadResourceId | Especifique el recurso de AAD para el que solicita autorización, por ejemplo, `https://management.core.windows.net`.| Sí |

**Ejemplo**

```json
{
    "name": "RESTLinkedService",
    "properties": {
        "type": "RestService",
        "typeProperties": {
            "url": "<REST endpoint e.g. https://www.example.com/>",
            "authenticationType": "ManagedServiceIdentity",
            "aadResourceId": "<AAD resource URL e.g. https://management.core.windows.net>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de REST. 

Para ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos y servicios vinculados](concepts-datasets-linked-services.md). 

Para copiar datos de REST, se admiten las siguientes propiedades:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del conjunto de datos debe establecerse en **RestResource**. | Sí |
| relativeUrl | Dirección URL relativa al recurso que contiene los datos. Cuando no se especifica la propiedad, solo se usa la dirección URL especificada en la definición del servicio vinculado. | Sin  |
| requestMethod | Método HTTP. Los valores permitidos son **Get** (valor predeterminado) y **Post**. | Sin  |
| additionalHeaders | Encabezados de solicitud HTTP adicionales. | Sin  |
| requestBody | Cuerpo de la solicitud HTTP. | Sin  |
| paginationRules | Las reglas de paginación para componer las solicitudes de página siguiente. Vea la sección [Compatibilidad con la paginación](#pagination-support) para obtener más información. | Sin  |

**Ejemplo 1: Mediante el método Get con la paginación**

```json
{
    "name": "RESTDataset",
    "properties": {
        "type": "RestResource",
        "linkedServiceName": {
            "referenceName": "<REST linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "relativeUrl": "<relative url>",
            "additionalHeaders": {
                "x-user-defined": "helloworld"
            },
            "paginationRules": {
                "AbsoluteUrl": "$.paging.next"
            }
        }
    }
}
```

**Ejemplo 2: Uso del método POST**

```json
{
    "name": "RESTDataset",
    "properties": {
        "type": "RestResource",
        "linkedServiceName": {
            "referenceName": "<REST linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "relativeUrl": "<relative url>",
            "requestMethod": "Post",
            "requestBody": "<body for POST REST request>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

En esta sección se proporciona una lista de las propiedades que admite el origen de REST.

Para ver una lista completa de las secciones y propiedades que hay disponibles para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md). 

### <a name="rest-as-source"></a>REST como origen

Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | DESCRIPCIÓN | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del origen de la actividad de copia debe establecerse en **RestSource**. | Sí |
| httpRequestTimeout | El tiempo de espera (el valor **TimeSpan**) para que la solicitud HTTP obtenga una respuesta. Este valor es el tiempo de espera para obtener una respuesta, no para leer los datos de la respuesta. El valor predeterminado es **00:01:40**.  | Sin  |
| requestInterval | El tiempo de espera antes de enviar la solicitud de página siguiente. El valor predeterminado es **00:00:01** |  Sin  |

**Ejemplo**

```json
"activities":[
    {
        "name": "CopyFromREST",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<REST input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "RestSource",
                "httpRequestTimeout": "00:01:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="pagination-support"></a>Compatibilidad con la paginación

Normalmente, la API REST limita su tamaño de carga de respuesta de una única solicitud a un número razonable; mientras que, para devolver gran cantidad de datos, divide el resultado en varias páginas y requiere que los autores de la llamada envíen solicitudes consecutivas para obtener la página siguiente del resultado. Por lo general, la solicitud para una página es dinámica y está compuesta por la información devuelta de la respuesta de página anterior.

Este conector REST genérico admite los siguientes patrones de paginación: 

* Dirección URL absoluta o relativa de la siguiente solicitud = valor de propiedad en el cuerpo de la respuesta actual
* Dirección URL absoluta o relativa de la siguiente solicitud = valor de encabezado en los encabezados de respuesta actual
* Parámetro de consulta de la siguiente solicitud = valor de propiedad en el cuerpo de la respuesta actual
* Parámetro de consulta de la siguiente solicitud = valor de encabezado en los encabezados de la respuesta actual
* Encabezado de la siguiente solicitud = valor de propiedad en el cuerpo de la respuesta actual
* Encabezado de la siguiente solicitud = valor de encabezado en los encabezados de la respuesta actual

**Las reglas de paginación** se definen como un diccionario en el conjunto de datos que contiene uno o más pares clave-valor que distinguen mayúsculas de minúsculas. La configuración se usará para generar la solicitud a partir de la segunda página. El conector detendrá la iteración cuando obtenga el código de estado HTTP 204 (sin contenido) o cualquiera de las expresiones JSONPath en "paginationRules" devuelva NULL.

**Claves admitidas** en las reglas de paginación:

| Clave | DESCRIPCIÓN |
|:--- |:--- |
| AbsoluteUrl | Indica la dirección URL para emitir la siguiente solicitud. Puede ser **dirección URL absoluta o relativa URL**. |
| QueryParameters.*request_query_parameter* O QueryParameters["request_query_parameter"] | El usuario define "request_query_parameter", que hace referencia a un nombre de parámetro de consulta en la siguiente dirección URL de solicitud HTTP. |
| Headers.*request_header* O Headers["request_header"] | El usuario define "request_header", que hace referencia a un nombre de encabezado en la siguiente solicitud HTTP. |

**Valores admitidos** en las reglas de paginación:

| Valor | DESCRIPCIÓN |
|:--- |:--- |
| Headers.*response_header* O Headers["response_header"] | El usuario define "response_header", que hace referencia a un nombre de encabezado en la respuesta HTTP actual, el valor que se usará para emitir la solicitud siguiente. |
| Una expresión JSONPath que empieza por "$" (que representa la raíz del cuerpo de respuesta) | El cuerpo de respuesta debe contener un único objeto JSON. La expresión JSONPath debe devolver un único valor primitivo, que se usará para emitir la solicitud siguiente. |

**Ejemplo:**

Facebook Graph API devuelve la respuesta en la siguiente estructura, en cuyo caso la dirección URL de la siguiente página se representa en ***paging.next***:

```json
{
    "data": [
        {
            "created_time": "2017-12-12T14:12:20+0000",
            "name": "album1",
            "id": "1809938745705498_1809939942372045"
        },
        {
            "created_time": "2017-12-12T14:14:03+0000",
            "name": "album2",
            "id": "1809938745705498_1809941802371859"
        },
        {
            "created_time": "2017-12-12T14:14:11+0000",
            "name": "album3",
            "id": "1809938745705498_1809941879038518"
        }
    ],
    "paging": {
        "cursors": {
            "after": "MTAxNTExOTQ1MjAwNzI5NDE=",
            "before": "NDMyNzQyODI3OTQw"
        },
        "previous": "https://graph.facebook.com/me/albums?limit=25&before=NDMyNzQyODI3OTQw",
        "next": "https://graph.facebook.com/me/albums?limit=25&after=MTAxNTExOTQ1MjAwNzI5NDE="
    }
}
```

La configuración del conjunto de datos REST correspondiente, especialmente la `paginationRules`, es como sigue:

```json
{
    "name": "MyFacebookAlbums",
    "properties": {
            "type": "RestResource",
            "typeProperties": {
                "relativeUrl": "albums",
                "paginationRules": {
                    "AbsoluteUrl": "$.paging.next"
                }
            },
            "linkedServiceName": {
                "referenceName": "MyRestService",
                "type": "LinkedServiceReference"
            }
    }
}
```

## <a name="export-json-response-as-is"></a>Exportación de la respuesta JSON tal cual

Puede usar este conector REST para exportar la respuesta de JSON de la API REST tal cual a varios almacenes basados en archivos. Para lograr esa copia independiente del esquema, omita la sección "structure" (también denominada *schema*) en el conjunto de datos y la asignación de esquemas en la actividad de copia.

## <a name="schema-mapping"></a>Asignación de esquemas

Para copiar datos desde el punto de conexión de REST en un receptor tabular, vea [asignación de esquemas](copy-activity-schema-and-type-mapping.md#schema-mapping).

## <a name="next-steps"></a>Pasos siguientes

Para ver una lista de los almacenes de datos que la actividad de copia admite como orígenes y receptores en Azure Data Factory, consulte los [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
