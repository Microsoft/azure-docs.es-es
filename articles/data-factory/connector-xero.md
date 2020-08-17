---
title: Copia de datos de Xero con Azure Data Factory
description: Obtenga información sobre cómo copiar datos desde Xero a almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/03/2020
ms.author: jingwang
ms.openlocfilehash: 14b3857211eca39ebe09a3a0752ca1d8eee17bc0
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87530000"
---
# <a name="copy-data-from-xero-using-azure-data-factory"></a>Copia de datos de Xero con Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de Xero. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector Xero es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos de Xero en cualquier almacén de datos receptor admitido. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector Xero admite lo siguiente:

- [Aplicación privada](https://developer.xero.com/documentation/getting-started/getting-started-guide), pero no aplicación pública de Xero.
- Todas las tablas de Xero (puntos de conexión de API), excepto "Reports" (Informes).
- Autenticación de OAuth 1.0 y OAuth 2.0.

Azure Data Factory proporciona un controlador integrado para habilitar la conectividad. Por lo tanto, no es necesario instalar manualmente ningún controlador mediante este conector.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector Xero.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Xero:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **Xero** | Sí |
| connectionProperties | Grupo de propiedades que define cómo conectarse a Xero. | Sí |
| ***En`connectionProperties`:*** | | |
| host | El punto de conexión del servidor de Xero (`api.xero.com`).  | Sí |
| authenticationType | Los valores permitidos son `OAuth_2.0` y `OAuth_1.0`. | Sí |
| consumerKey | Clave de consumidor asociada a la aplicación de Xero. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| privateKey | La clave privada del archivo .pem que se generó para la aplicación privada de Xero; consulte [Create a public/private key pair](https://developer.xero.com/documentation/auth-and-limits/create-publicprivate-key) (Creación de un par de claves pública y privada). Asegúrese de **generar el archivo privatekey.pem con numbits de 512** con `openssl genrsa -out privatekey.pem 512`; 1024 no se admite. Incluya todo el texto del archivo .pem, así como los finales de línea Unix (\n). Vea el ejemplo a continuación.<br/>Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| tenantId | Identificador de inquilino asociado a la aplicación Xero. Aplicable para la autenticación de OAuth 2.0.<br>Aprenda a obtener el identificador de inquilino en la sección [Comprobación de los inquilinos a los que tiene autorizado el acceso](https://developer.xero.com/documentation/oauth2/auth-flow). | Sí para la autenticación de OAuth 2.0 |
| refreshToken | Token de actualización de OAuth 2.0 asociado a la aplicación Xero, que se usa para actualizar el token de acceso cuando este expira. Aplicable para la autenticación de OAuth 2.0. Aprenda a obtener el token de actualización de [este artículo](https://developer.xero.com/documentation/oauth2/auth-flow).<br>El token de actualización no tiene fecha de expiración. Para obtener un token de actualización, debe solicitar el [ámbito offline_access](https://developer.xero.com/documentation/oauth2/scopes).<br/>Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí para la autenticación de OAuth 2.0 |
| useEncryptedEndpoints | Especifica si los puntos de conexión de origen de datos se cifran mediante HTTPS. El valor predeterminado es true.  | No |
| useHostVerification | Especifica si se requiere que el nombre de host del certificado del servidor coincida con el nombre de host del servidor al conectarse a través de TLS. El valor predeterminado es true.  | No |
| usePeerVerification | Especifica si se debe verificar la identidad del servidor al conectarse a través de TLS. El valor predeterminado es true.  | No |

**Ejemplo: autenticación de OAuth 2.0**

```json
{
    "name": "XeroLinkedService",
    "properties": {
        "type": "Xero",
        "typeProperties": {
            "connectionProperties": { 
                "host": "api.xero.com",
                "authenticationType":"OAuth_2.0", 
                "consumerKey": {
                    "type": "SecureString",
                    "value": "<consumer key>"
                },
                "privateKey": {
                    "type": "SecureString",
                    "value": "<private key>"
                },
                "tenantId": "<tenant ID>", 
                "refreshToken": {
                    "type": "SecureString",
                    "value": "<refresh token>"
                }, 
                "useEncryptedEndpoints": true, 
                "useHostVerification": true, 
                "usePeerVerification": true
            }            
        }
    }
}
```

**Ejemplo: autenticación de OAuth 1.0**

```json
{
    "name": "XeroLinkedService",
    "properties": {
        "type": "Xero",
        "typeProperties": {
            "connectionProperties": {
                "host": "api.xero.com", 
                "authenticationType":"OAuth_1.0", 
                "consumerKey": {
                    "type": "SecureString",
                    "value": "<consumer key>"
                },
                "privateKey": {
                    "type": "SecureString",
                    "value": "<private key>"
                }, 
                "useEncryptedEndpoints": true,
                "useHostVerification": true,
                "usePeerVerification": true
            }
        }
    }
}
```

**Valor de clave privada de ejemplo:**

Incluya todo el texto del archivo .pem, así como los finales de línea Unix (\n).

```
"-----BEGIN RSA PRIVATE KEY-----\nMII***************************************************P\nbu****************************************************s\nU/****************************************************B\nA*****************************************************W\njH****************************************************e\nsx*****************************************************l\nq******************************************************X\nh*****************************************************i\nd*****************************************************s\nA*****************************************************dsfb\nN*****************************************************M\np*****************************************************Ly\nK*****************************************************Y=\n-----END RSA PRIVATE KEY-----"
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Xero.

Para copiar datos de Xero, establezca la propiedad type del conjunto de datos en **XeroObject**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **XeroObject** | Sí |
| tableName | Nombre de la tabla. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "XeroDataset",
    "properties": {
        "type": "XeroObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Xero linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen de Xero.

### <a name="xero-as-source"></a>Xero como origen

Para copiar datos de Xero, establezca el tipo de origen de la actividad de copia en **XeroSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **XeroSource** | Sí |
| Query | Use la consulta SQL personalizada para leer los datos. Por ejemplo: `"SELECT * FROM Contacts"`. | No (si se especifica "tableName" en el conjunto de datos) |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromXero",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Xero input dataset name>",
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
                "type": "XeroSource",
                "query": "SELECT * FROM Contacts"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Tenga en cuenta lo siguiente al especificar la consulta de Xero:

- Las tablas con elementos complejos se dividirán en varias tablas. Por ejemplo, las transacciones bancarias tienen una estructura de datos compleja "LineItems", así que los datos de la transacción bancaria se asignan a las tablas `Bank_Transaction` y `Bank_Transaction_Line_Items`, con la clave externa `Bank_Transaction_ID` para vincularlas.

- Los datos de Xero están disponibles a través de dos esquemas: `Minimal` (predeterminado) y `Complete`. El esquema Complete contiene tablas de llamadas de requisitos previos que necesitan datos adicionales (por ejemplo, la columna ID) antes de realizar la consulta deseada.

Las siguientes tablas tienen la misma información en los esquemas Minimal y Complete. Para reducir el número de llamadas API, use un esquema Minimal (predeterminado).

- Bank_Transactions
- Contact_Groups 
- Contactos 
- Contacts_Sales_Tracking_Categories 
- Contacts_Phones 
- Contacts_Addresses 
- Contacts_Purchases_Tracking_Categories 
- Credit_Notes 
- Credit_Notes_Allocations 
- Expense_Claims 
- Expense_Claim_Validation_Errors
- Facturas 
- Invoices_Credit_Notes
- Invoices_ Prepayments 
- Invoices_Overpayments 
- Manual_Journals 
- Overpayments 
- Overpayments_Allocations 
- Prepayments 
- Prepayments_Allocations 
- Receipts 
- Receipt_Validation_Errors 
- Tracking_Categories

Las siguientes tablas solo se pueden consultar con el esquema Complete:

- Complete.Bank_Transaction_Line_Items 
- Complete.Bank_Transaction_Line_Item_Tracking 
- Complete.Contact_Group_Contacts 
- Complete.Contacts_Contact_ Persons 
- Complete.Credit_Note_Line_Items 
- Complete.Credit_Notes_Line_Items_Tracking 
- Complete.Expense_Claim_ Payments 
- Complete.Expense_Claim_Receipts 
- Complete.Invoice_Line_Items 
- Complete.Invoices_Line_Items_Tracking
- Complete.Manual_Journal_Lines 
- Complete.Manual_Journal_Line_Tracking 
- Complete.Overpayment_Line_Items 
- Complete.Overpayment_Line_Items_Tracking 
- Complete.Prepayment_Line_Items 
- Complete.Prepayment_Line_Item_Tracking 
- Complete.Receipt_Line_Items 
- Complete.Receipt_Line_Item_Tracking 
- Complete.Tracking_Category_Options

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Pasos siguientes
Para obtener una lista de los almacenes de datos que admite la actividad de copia, consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats).
