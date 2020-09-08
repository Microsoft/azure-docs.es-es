---
title: Copia de datos de un origen REST mediante Azure Data Factory
description: Obtenga información sobre cómo copiar datos desde un origen REST, en la nube o en un entorno local, a almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: jingwang
ms.openlocfilehash: c0a64c0a9653bd274e9298401163ad7abc1af99f
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852300"
---
# <a name="copy-data-from-a-rest-endpoint-by-using-azure-data-factory"></a>Copia de datos desde un punto de conexión REST mediante Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde un punto de conexión REST. El artículo se basa en [Actividad de copia en Azure Data Factory](copy-activity-overview.md), en el que se ofrece información general acerca de la actividad de copia.

Las diferencias entre este conector REST, el [conector HTTP](connector-http.md) y el [conector de tabla web](connector-web-table.md) son:

- El **conector REST** admite específicamente la copia de datos desde API de RESTful; 
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

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que puede usar para definir entidades de Data Factory específicas del conector REST.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de REST:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **RestService**. | Sí |
| url | La dirección URL base del servicio REST. | Sí |
| enableServerCertificateValidation | Si se debe validar el certificado TLS/SSL del lado servidor al conectarse al punto de conexión. | No<br /> (El valor predeterminado es: **true**) |
| authenticationType | El tipo de autenticación usado para conectarse al servicio REST. Los valores que se permiten son: **Anónima**, **Básica**, **AadServicePrincipal** y **ManagedServiceIdentity**. Haga referencia a las siguientes secciones correspondientes para obtener más información sobre propiedades y ejemplos, respectivamente. | Sí |
| connectVia | Instancia de [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, esta propiedad se usará Azure Integration Runtime. |No |

### <a name="use-basic-authentication"></a>Uso de la autenticación básica

Establezca la propiedad **authenticationType** en **Basic**. Además de las propiedades genéricas descritas en las secciones anteriores, especifique las siguientes:

| Propiedad | Descripción | Obligatorio |
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

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| servicePrincipalId | Especifique el identificador de cliente de la aplicación de Azure Active Directory. | Sí |
| servicePrincipalKey | Especifique la clave de la aplicación de Azure Active Directory. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**. | Sí |
| tenant | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarla, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí |
| aadResourceId | Especifique el recurso de AAD para el que solicita autorización, por ejemplo, `https://management.core.windows.net`.| Sí |
| azureCloudType | Para la autenticación de la entidad de servicio, especifique el tipo de entorno de nube de Azure en el que está registrada la aplicación de AAD. <br/> Los valores permitidos son **AzurePublic**, **AzureChina**, **AzureUsGovernment** y **AzureGermany**. De forma predeterminada, se usa el entorno de nube de la factoría de datos. | No |

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

### <a name="use-managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a> Uso de identidades administradas para la autenticación de recursos de Azure

Establezca la propiedad **authenticationType** en **ManagedServiceIdentity**. Además de las propiedades genéricas descritas en las secciones anteriores, especifique las siguientes:

| Propiedad | Descripción | Obligatorio |
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

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del conjunto de datos debe establecerse en **RestResource**. | Sí |
| relativeUrl | Dirección URL relativa al recurso que contiene los datos. Cuando no se especifica la propiedad, solo se usa la dirección URL especificada en la definición del servicio vinculado. El conector HTTP copia los datos de la dirección URL combinada: `[URL specified in linked service]/[relative URL specified in dataset]`. | No |

Si estaba configurando `requestMethod`, `additionalHeaders`, `requestBody` y `paginationRules` en el conjunto de datos, todavía se admite tal cual, aunque se aconseja usar el nuevo modelo en la fuente de actividad en el futuro.

**Ejemplo**:

```json
{
    "name": "RESTDataset",
    "properties": {
        "type": "RestResource",
        "typeProperties": {
            "relativeUrl": "<relative url>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<REST linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

En esta sección se proporciona una lista de las propiedades que admite el origen de REST.

Para ver una lista completa de las secciones y propiedades que hay disponibles para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md). 

### <a name="rest-as-source"></a>REST como origen

Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del origen de la actividad de copia debe establecerse en **RestSource**. | Sí |
| requestMethod | Método HTTP. Los valores permitidos son **Get** (valor predeterminado) y **Post**. | No |
| additionalHeaders | Encabezados de solicitud HTTP adicionales. | No |
| requestBody | Cuerpo de la solicitud HTTP. | No |
| paginationRules | Las reglas de paginación para componer las solicitudes de página siguiente. Vea la sección [Compatibilidad con la paginación](#pagination-support) para obtener más información. | No |
| httpRequestTimeout | El tiempo de espera (el valor **TimeSpan**) para que la solicitud HTTP obtenga una respuesta. Este valor es el tiempo de espera para obtener una respuesta, no para leer los datos de la respuesta. El valor predeterminado es **00:01:40**.  | No |
| requestInterval | El tiempo de espera antes de enviar la solicitud de página siguiente. El valor predeterminado es **00:00:01** |  No |

>[!NOTE]
>El conector REST ignora cualquier encabezado "Accept" que se especifique en `additionalHeaders`. Como el conector REST solo admite la respuesta en JSON, generará automáticamente un encabezado de `Accept: application/json`.

**Ejemplo 1: Mediante el método Get con la paginación**

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
                "additionalHeaders": {
                    "x-user-defined": "helloworld"
                },
                "paginationRules": {
                    "AbsoluteUrl": "$.paging.next"
                },
                "httpRequestTimeout": "00:01:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Ejemplo 2: Uso del método POST**

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
                "requestMethod": "Post",
                "requestBody": "<body for POST REST request>",
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

Normalmente, la API de REST limita su tamaño de carga de respuesta de una única solicitud a un número razonable; mientras que, para devolver gran cantidad de datos, divide el resultado en varias páginas y requiere que los autores de la llamada envíen solicitudes consecutivas para obtener la página siguiente del resultado. Por lo general, la solicitud para una página es dinámica y está compuesta por la información devuelta de la respuesta de página anterior.

Este conector REST genérico admite los siguientes patrones de paginación: 

* Dirección URL absoluta o relativa de la siguiente solicitud = valor de propiedad en el cuerpo de la respuesta actual
* Dirección URL absoluta o relativa de la siguiente solicitud = valor de encabezado en los encabezados de la respuesta actual
* Parámetro de consulta de la siguiente solicitud = valor de propiedad en el cuerpo de la respuesta actual
* Parámetro de consulta de la siguiente solicitud = valor de encabezado en los encabezados de la respuesta actual
* Encabezado de la siguiente solicitud = valor de propiedad en el cuerpo de la respuesta actual
* Encabezado de la siguiente solicitud = valor de encabezado en los encabezados de la respuesta actual

Las **reglas de paginación** se definen como un diccionario en el conjunto de datos que contiene uno o más pares clave-valor que distinguen mayúsculas de minúsculas. La configuración se usará para generar la solicitud a partir de la segunda página. El conector detendrá la iteración cuando obtenga el código de estado HTTP 204 (sin contenido) o cualquiera de las expresiones JSONPath en "paginationRules" devuelva NULL.

**Claves admitidas** en las reglas de paginación:

| Clave | Descripción |
|:--- |:--- |
| AbsoluteUrl | Indica la dirección URL para emitir la siguiente solicitud. Puede ser una **dirección URL absoluta o relativa**. |
| QueryParameters.*request_query_parameter* O QueryParameters["request_query_parameter"] | El usuario define "request_query_parameter", que hace referencia a un nombre de parámetro de consulta en la siguiente dirección URL de solicitud HTTP. |
| Headers.*request_header* O Headers["request_header"] | El usuario define "request_header", que hace referencia a un nombre de encabezado en la siguiente solicitud HTTP. |

**Valores admitidos** en las reglas de paginación:

| Value | Descripción |
|:--- |:--- |
| Headers.*response_header* O Headers["response_header"] | El usuario define "response_header", que hace referencia a un nombre de encabezado en la respuesta HTTP actual, el valor que se usará para emitir la solicitud siguiente. |
| Una expresión JSONPath que empieza por "$" (que representa la raíz del cuerpo de respuesta) | El cuerpo de respuesta debe contener un único objeto JSON. La expresión JSONPath debe devolver un único valor primitivo, que se usará para emitir la solicitud siguiente. |

**Ejemplo**:

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
"typeProperties": {
    "source": {
        "type": "RestSource",
        "paginationRules": {
            "AbsoluteUrl": "$.paging.next"
        },
        ...
    },
    "sink": {
        "type": "<sink type>"
    }
}
```

## <a name="use-oauth"></a>Uso de OAuth
En esta sección se describe cómo usar una plantilla de solución para copiar datos del conector REST en Azure Data Lake Storage en formato JSON mediante OAuth. 

### <a name="about-the-solution-template"></a>Acerca de la plantilla de solución

La plantilla contiene dos actividades:
- La actividad **web** recupera el token de portador y, a continuación, lo pasa a la actividad de copia posterior como autorización.
- La actividad de **copia** se encarga de copiar los datos de REST en Azure Data Lake Storage.

La plantilla define dos parámetros:
- **SinkContainer** es la ruta de acceso de la carpeta raíz en la que se copian los datos en Azure Data Lake Storage. 
- **SinkDirectory** es la ruta de acceso del directorio en la raíz en el que se copian los datos en Azure Data Lake Storage. 

### <a name="how-to-use-this-solution-template"></a>Uso de esta plantilla de solución

1. Vaya a la plantilla **Copiar desde REST o HTTP con OAuth**. Cree una nueva conexión para la conexión de origen. 
    ![Creación de conexiones](media/solution-template-copy-from-rest-or-http-using-oauth/source-connection.png)

    A continuación se muestran los pasos principales para configurar el nuevo servicio vinculado (REST):
    
     1. En **URL base**, especifique el parámetro de dirección URL para su servicio REST de origen. 
     2. En **Tipo de autenticación**, elija *Anónima*.
        ![Nueva conexión REST](media/solution-template-copy-from-rest-or-http-using-oauth/new-rest-connection.png)

2. Cree una nueva conexión para la conexión de destino.  
    ![Conexión de Gen2 nueva](media/solution-template-copy-from-rest-or-http-using-oauth/destination-connection.png)

3. Seleccione **Usar esta plantilla**.
    ![Usar esta plantilla](media/solution-template-copy-from-rest-or-http-using-oauth/use-this-template.png)

4. Verá la canalización creada tal y como se muestra en el ejemplo siguiente:  ![Canalización](media/solution-template-copy-from-rest-or-http-using-oauth/pipeline.png)

5. Seleccione la actividad **web**. En **Configuración**, especifique la **URL**, **Método**, **Encabezados** y **Cuerpo** correspondientes para recuperar el token de portador OAuth de la API de inicio de sesión del servicio desde el que quiere copiar los datos. El marcador de posición de la plantilla muestra un ejemplo de OAuth de Azure Active Directory (AAD). Nota: la autenticación de AAD es compatible de forma nativa con el conector REST; este es solo un ejemplo de flujo de OAuth. 

    | Propiedad | Descripción |
    |:--- |:--- |:--- |
    | URL |Especifique la dirección URL de la que se recuperará el token de portador OAuth. p. ej., en este ejemplo es https://login.microsoftonline.com/microsoft.onmicrosoft.com/oauth2/token |. 
    | Método | Método HTTP. Los valores permitidos son **POST** y **GET**. | 
    | encabezados | El usuario define el encabezado, que hace referencia a un nombre de encabezado en la solicitud HTTP. | 
    | Body | Cuerpo de la solicitud HTTP. | 

    ![Canalización](media/solution-template-copy-from-rest-or-http-using-oauth/web-settings.png)

6. En la actividad **Copiar datos**, seleccione la pestaña *Origen* y verá que el token de portador (access_token) recuperado del paso anterior se pasará a la actividad de copia de datos como **autorización** en los encabezados adicionales. Confirme la configuración de las siguientes propiedades antes de iniciar una ejecución de canalización.

    | Propiedad | Descripción |
    |:--- |:--- |:--- | 
    | Método de solicitud | Método HTTP. Los valores permitidos son **Get** (valor predeterminado) y **Post**. | 
    | Encabezados adicionales | Encabezados de solicitud HTTP adicionales.| 

   ![Autenticación para la copia del origen](media/solution-template-copy-from-rest-or-http-using-oauth/copy-data-settings.png)

7. Seleccione **Depurar**, escriba los **parámetros** y, a continuación, seleccione **Finalizar**.
   ![Ejecución de la canalización](media/solution-template-copy-from-rest-or-http-using-oauth/pipeline-run.png) 

8. Cuando la ejecución de la canalización se complete correctamente, verá un resultado similar al ejemplo siguiente: ![Resultado de la ejecución de la canalización](media/solution-template-copy-from-rest-or-http-using-oauth/run-result.png) 

9. Haga clic en el icono de "Salida" de WebActivity en la columna **Acciones** para ver el access_token devuelto por el servicio.

   ![Salida del token](media/solution-template-copy-from-rest-or-http-using-oauth/token-output.png) 

10. Haga clic en el icono de "Entrada" de CopyActivity en la columna **Acciones** para ver que el access_token recuperado por WebActivity se pasa a CopyActivity para la autenticación. 

    ![Entrada del token](media/solution-template-copy-from-rest-or-http-using-oauth/token-input.png)
        
    >[!CAUTION] 
    >Para evitar que el token se registre como texto sin formato, habilite la opción "Salida segura" en la actividad web y "Entrada segura" en la actividad de copia.


## <a name="export-json-response-as-is"></a>Exportación de la respuesta JSON tal cual

Puede usar este conector REST para exportar la respuesta de JSON de la API REST tal cual a varios almacenes basados en archivos. Para lograr esa copia independiente del esquema, omita la sección “structure” (estructura, también denominada *schema*) en el conjunto de datos y la asignación de esquemas en la actividad de copia.

## <a name="schema-mapping"></a>Asignación de esquemas

Para copiar datos desde el punto de conexión de REST en un receptor tabular, vea [asignación de esquemas](copy-activity-schema-and-type-mapping.md#schema-mapping).

## <a name="next-steps"></a>Pasos siguientes

Para ver una lista de los almacenes de datos que la actividad de copia admite como orígenes y receptores en Azure Data Factory, consulte los [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
