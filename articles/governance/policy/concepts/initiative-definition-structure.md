---
title: Detalles de la estructura de definición de iniciativa
description: Describe cómo se usan las definiciones de iniciativa de directiva para agrupar definiciones de directiva para la implementación en los recursos de Azure de la organización.
ms.date: 08/17/2020
ms.topic: conceptual
ms.openlocfilehash: d7b4adf15193e2cd1b9e516a04c7c989dc442ee9
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89048506"
---
# <a name="azure-policy-initiative-definition-structure"></a>Estructura de definición de iniciativa de Azure Policy

Las iniciativas le permiten agrupan varias definiciones de directivas relacionadas para simplificar las asignaciones y la administración, porque se trabaja con un grupo como un elemento único. Por ejemplo, puede agrupar las definiciones de directivas de etiquetado relacionadas en una sola iniciativa. En lugar de asignar individualmente cada directiva, la aplica.

Para crear una definición de iniciativa de directiva se usa JSON. La definición de iniciativa de directiva contiene elementos para:

- nombre para mostrar
- description
- metadata
- parámetros
- definiciones de directiva
- grupos de directivas (esta propiedad forma parte de la [característica Cumplimiento reglamentario (versión preliminar)](./regulatory-compliance.md))

En el ejemplo siguiente se muestra cómo crear una iniciativa para controlar dos etiquetas: `costCenter` y `productName`. Usa dos directivas integradas para aplicar el valor de etiqueta predeterminado.

```json
{
    "properties": {
        "displayName": "Billing Tags Policy",
        "policyType": "Custom",
        "description": "Specify cost Center tag and product name tag",
        "metadata": {
            "version": "1.0.0",
            "category": "Tags"
        },
        "parameters": {
            "costCenterValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for Cost Center tag"
                },
                "defaultValue": "DefaultCostCenter"
            },
            "productNameValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for product Name tag"
                },
                "defaultValue": "DefaultProduct"
            }
        },
        "policyDefinitions": [{
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            }
        ]
    }
}
```

Los patrones y elementos integrados de Azure Policy se encuentran en [Ejemplos de Azure Policy](../samples/index.md).

## <a name="metadata"></a>Metadatos

La propiedad `metadata` opcional almacena información sobre la definición de iniciativa de directiva.
Los clientes pueden definir las propiedades y los valores útiles para su organización en `metadata`. Aun así, hay algunas propiedades _comunes_ que se usan en Azure Policy y los elementos integrados.

### <a name="common-metadata-properties"></a>Propiedades de metadatos comunes

- `version` (cadena): realiza el seguimiento de los detalles sobre la versión del contenido de una definición de iniciativa de directiva.
- `category` (cadena): determina en qué categoría de Azure Portal se muestra la definición de directiva.

  > [!NOTE]
  > En el caso de una iniciativa de [Cumplimiento reglamentario](./regulatory-compliance.md), `category` debe ser **Cumplimiento reglamentario**.

- `preview` (booleano): marca true o false si la definición de iniciativa de directiva es _versión preliminar_.
- `deprecated` (booleano): marca true o false si la definición de iniciativa de directiva está marcada como _en desuso_.

> [!NOTE]
> El servicio Azure Policy usa las propiedades `version`, `preview` y `deprecated` para transmitir el nivel de cambio a una definición o iniciativa de directiva integradas y el estado. El formato de `version` es: `{Major}.{Minor}.{Patch}`. Determinados estados, como _en desuso_ o _versión preliminar_, están anexados a la propiedad `version` o están en otra propiedad como **booleano**. Para obtener más información sobre la forma en que Azure Policy crea versiones los elementos integrados, vea [Control de versiones integradas](https://github.com/Azure/azure-policy/blob/master/built-in-policies/README.md).

## <a name="parameters"></a>Parámetros

Los parámetros ayudan a simplificar la administración de directivas mediante la reducción del número de definiciones de directiva. Piense en los parámetros como si fueran los campos de un formulario: `name`, `address`, `city`, `state`. Estos parámetros no cambian, pero sí sus valores en función del individuo que rellena el formulario.
Los parámetros funcionan del mismo modo al crear iniciativas de directiva. Si se incluyen parámetros en una definición de iniciativa de directiva, se puede volver a usar ese parámetro en las directivas incluidas.

> [!NOTE]
> Una vez que se asigna una iniciativa, no se pueden modificar los parámetros de nivel de iniciativa. Por eso, se recomienda establecer un **defaultValue** al definir el parámetro.

### <a name="parameter-properties"></a>Propiedades del parámetro

Un parámetro tiene las siguientes propiedades que se usan en la definición de iniciativa de directiva:

- `name`: El nombre del parámetro. Lo utiliza la función de la implementación `parameters` dentro de la regla de directiva. Para más información, consulte [Uso de un valor de parámetro](#passing-a-parameter-value-to-a-policy-definition).
- `type`: Determina si el parámetro es **string**, **array**, **object**, **boolean**, **integer**, **float** o **datetime**.
- `metadata`: Define las subpropiedades que usa principalmente Azure Portal para mostrar información intuitiva:
  - `description`: La explicación de para qué se usa el parámetro. Puede utilizarse para proporcionar ejemplos de valores aceptables.
  - `displayName`: El nombre descriptivo que se muestra en el portal para el parámetro.
  - `strongType`: (Opcional) Se usa al asignar la definición de directiva mediante el portal. Proporciona una lista que tiene en cuenta el contexto. Para más información, consulte [strongType](#strongtype).
- `defaultValue`: (Opcional) Establece el valor del parámetro en una asignación, si no se especifica ningún valor.
- `allowedValues`: (Opcional) Proporciona una matriz de los valores que acepta el parámetro durante la asignación.

Por ejemplo, puede establecer una definición de iniciativa de directiva para limitar las ubicaciones de los recursos en las distintas definiciones de directiva incluidas. Un parámetro para esa definición de iniciativa de directiva podría ser **allowedLocations**. Después, el parámetro estará disponible para cada definición de directiva incluida y se definirá durante la asignación de la iniciativa de directiva.

```json
"parameters": {
    "init_allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": [ "westus2" ],
        "allowedValues": [
            "eastus2",
            "westus2",
            "westus"
        ]
    }
}
```

### <a name="passing-a-parameter-value-to-a-policy-definition"></a>Transferencia de un valor de parámetro a una definición de directiva

Debe declarar qué parámetros de iniciativa que se pasan a qué definiciones de directiva incluidas en la matriz [policyDefinitions](#policy-definitions) de la definición de iniciativa. Aunque el nombre del parámetro puede ser el mismo, el uso de nombres en las iniciativas diferentes a los de las definiciones de directiva simplifica la legibilidad del código.

Por ejemplo, el parámetro de iniciativa **init_allowedLocations** definido anteriormente se puede pasar a varias definiciones de directiva incluidas y sus parámetros (**sql_locations** y **vm_locations**) como se indica a continuación:

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0ec8fc28-d5b7-4603-8fec-39044f00a92b",
        "policyDefinitionReferenceId": "allowedLocationsSQL",
        "parameters": {
            "sql_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/aa09bd0f-aa5f-4343-b6ab-a33a6a6304f3",
        "policyDefinitionReferenceId": "allowedLocationsVMs",
        "parameters": {
            "vm_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    }
]
```

En este ejemplo se hace referencia al parámetro **init_allowedLocations** que se mostró en las [ propiedades del parámetro](#parameter-properties).

### <a name="strongtype"></a>strongType

Dentro de la propiedad `metadata`, puede usar **strongType** para proporcionar una lista de opciones de selección múltiple en Azure Portal. **strongType** puede ser un _tipo de recurso_ compatible o un valor permitido. Para determinar si un _tipo de recurso_ es válido para **strongType**, use [Get-AzResourceProvider](/powershell/module/az.resources/get-azresourceprovider).

Se admiten algunos tipos de recursos que no devuelve **Get-AzResourceProvider**. Esos tipos de recursos son los siguientes:

- `Microsoft.RecoveryServices/vaults/backupPolicies`

Los valores admitidos para **strongType** que no son tipo de recurso son:

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`

## <a name="policy-definitions"></a>Definiciones de directiva

La parte `policyDefinitions` de la definición de iniciativa es un elemento _array_ de las definiciones de directiva existentes que se incluyen en la iniciativa. Como se indicó en [Transferencia de un valor de parámetro a una definición de directiva](#passing-a-parameter-value-to-a-policy-definition), esta propiedad es donde se pasan los [parámetros de iniciativa](#parameters) a la definición de directiva.

### <a name="policy-definition-properties"></a>Propiedades de definición de directiva

Cada elemento _array_ que representa una definición de directiva tiene las siguientes propiedades:

- `policyDefinitionId` (cadena): identificador de la definición de directiva personalizada o integrada que se va a incluir.
- `policyDefinitionReferenceId` (cadena): nombre corto para la definición de directiva incluida.
- `parameters`: (opcional) pares de nombre-valor para pasar un parámetro de iniciativa a la definición de directiva incluida como una propiedad en esa definición de directiva. Para obtener más información, vea [Parámetros](#parameters).
- `groupNames` (matriz de cadenas): (opcional) grupo del que es miembro la definición de directiva. Para obtener más información, vea [Grupos de directivas](#policy-definition-groups).

Este es un ejemplo de `policyDefinitions` con dos definiciones de directiva incluidas, a las que se pasa el mismo parámetro de iniciativa:

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0ec8fc28-d5b7-4603-8fec-39044f00a92b",
        "policyDefinitionReferenceId": "allowedLocationsSQL",
        "parameters": {
            "sql_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/aa09bd0f-aa5f-4343-b6ab-a33a6a6304f3",
        "policyDefinitionReferenceId": "allowedLocationsVMs",
        "parameters": {
            "vm_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    }
]
```

## <a name="policy-definitions-groups-preview"></a><a name="policy-definition-groups"></a>Grupos de definiciones de directiva (versión preliminar)

Como parte de la característica [Cumplimiento reglamentario](./regulatory-compliance.md) (versión preliminar) de Azure Policy, es posible agrupar definiciones de directiva en una definición de iniciativa. Esta información se define en la propiedad _array_ `policyDefinitionGroups`. Estas agrupaciones tienen detalles adicionales, como el **control** y el **dominio de cumplimiento**, para los que la definición de directiva proporciona cobertura.
Se pueden encontrar más detalles de las agrupaciones en un objeto **policyMetadata** que crea Microsoft. Para obtener más información, vea [Objetos de metadatos](#metadata-objects).

### <a name="policy-definition-groups-parameters"></a>Parámetros de grupos de definiciones de directiva

Cada elemento _array_ de `policyDefinitionGroups` debe tener las siguientes propiedades:

- `name` (cadena) \[obligatorio\]: nombre corto del **control**. El valor de esta propiedad lo usa `groupNames` en `policyDefinitions`.
- `category` (cadena): **dominio de cumplimiento** del control.
- `displayName` (cadena): nombre descriptivo del **control**. Lo usa el portal.
- `description` (cadena): descripción de lo que hace el **control**.
- `additionalMetadataId` (cadena): ubicación del objeto [policyMetadata](#metadata-objects) que contiene detalles adicionales sobre el **control** y el **dominio de cumplimiento**.

  > [!NOTE]
  > Los clientes pueden apuntar a un objeto [policyMetadata](#metadata-objects) existente. Aun así, estos objetos son de _solo lectura_ y únicamente los crea Microsoft.

Aquí se muestra un ejemplo del aspecto que tendría la propiedad `policyDefinitionGroups` de la definición de iniciativa integrada de NIST:

```json
"policyDefinitionGroups": [
    {
        "name": "NIST_SP_800-53_R4_AC-1",
        "additionalMetadataId": "/providers/Microsoft.PolicyInsights/policyMetadata/NIST_SP_800-53_R4_AC-1"
    }
]
```

### <a name="metadata-objects"></a>Objetos de metadatos

Los componentes integrados de cumplimiento reglamentario que crea Microsoft contienen información adicional sobre cada control.
Esta información es la siguiente:

- Se muestra en Azure Portal en la información general de un **control** en una iniciativa de cumplimiento reglamentario.
- Disponible a través de la API de REST. Vea el proveedor de recursos `Microsoft.PolicyInsights` y el [grupo de operaciones policyMetadata](/rest/api/policy-insights/policymetadata/getresource).
- Disponible mediante la CLI de Azure. Vea el comando [az policy metadata](/cli/azure/policy/metadata).

> [!IMPORTANT]
> Los objetos de metadatos para el cumplimiento reglamentario son de _solo lectura_ y no los pueden crear los clientes.

Los metadatos de una agrupación de directivas tienen la siguiente información en el nodo `properties`:

- `metadataId`: **identificador de control** con el que se relaciona la agrupación.
- `category` (obligatorio): **dominio de cumplimiento** al que pertenece el **control**.
- `title` (obligatorio): nombre descriptivo del **identificador del control**.
- `owner` (obligatorio): identifica quién tiene la responsabilidad del control en Azure: _Cliente_, _Microsoft_ o _Compartido_.
- `description`: información adicional sobre el control.
- `requirements`: detalles sobre la responsabilidad de la implementación del control.
- `additionalContentUrl`: vínculo a más información sobre el control. Esta propiedad suele ser un vínculo a la sección de documentación que trata sobre este control en el estándar de cumplimiento.

Más abajo se muestra un ejemplo del objeto **policyMetadata**. Los metadatos de este ejemplo pertenecen al control _NIST SP 800-53 R4 AC-1_.

```json
{
  "properties": {
    "metadataId": "NIST SP 800-53 R4 AC-1",
    "category": "Access Control",
    "title": "Access Control Policy and Procedures",
    "owner": "Shared",
    "description": "**The organization:**    \na. Develops, documents, and disseminates to [Assignment: organization-defined personnel or roles]:  \n1. An access control policy that addresses purpose, scope, roles, responsibilities, management commitment, coordination among organizational entities, and compliance; and  \n2. Procedures to facilitate the implementation of the access control policy and associated access controls; and  \n  
\nb. Reviews and updates the current:  \n1. Access control policy [Assignment: organization-defined frequency]; and  \n2. Access control procedures [Assignment: organization-defined frequency].",
    "requirements": "**a.**  The customer is responsible for developing, documenting, and disseminating access control policies and procedures. The customer access control policies and procedures address access to all customer-deployed resources and customer system access (e.g., access to customer-deployed virtual machines, access to customer-built applications).  \n**b.**  The customer is responsible for reviewing and updating access control policies and procedures in accordance with FedRAMP requirements.",
    "additionalContentUrl": "https://nvd.nist.gov/800-53/Rev4/control/AC-1"
  },
  "id": "/providers/Microsoft.PolicyInsights/policyMetadata/NIST_SP_800-53_R4_AC-1",
  "name": "NIST_SP_800-53_R4_AC-1",
  "type": "Microsoft.PolicyInsights/policyMetadata"
}
```

## <a name="next-steps"></a>Pasos siguientes

- Vea la [estructura de definición](./definition-structure.md).
- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Vea la [Descripción de los efectos de directivas](effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
