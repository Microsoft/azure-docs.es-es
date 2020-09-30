---
title: Archivo CreateUiDefinition.jso para el panel del portal
description: Describe cómo crear definiciones de interfaz de usuario para Azure Portal. Se usa al definir Azure Managed Applications.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: tomfitz
ms.openlocfilehash: 327fa1d7eb73d8e65bb4f81c1dff0fe2bec2913b
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89319581"
---
# <a name="createuidefinitionjson-for-azure-managed-applications-create-experience"></a>CreateUiDefinition.json para la experiencia de creación de aplicaciones administradas de Azure

En este documento se presentan los conceptos principales del archivo **createUiDefinition.json**. Azure Portal usa este archivo para generar la interfaz de usuario al crear una aplicación administrada.

La plantilla es la siguiente:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Azure.CreateUIDef",
    "version": "0.1.2-preview",
    "parameters": {
        "config": {
            "isWizard": false,
            "basics": { }
        },
        "basics": [ ],
        "steps": [ ],
        "outputs": { },
        "resourceTypes": [ ]
    }
}
```

`CreateUiDefinition` siempre contiene tres propiedades:

* handler
* version
* parámetros

Handler siempre debería ser `Microsoft.Azure.CreateUIDef`, y la última versión admitida es `0.1.2-preview`.

El esquema de la propiedad parameters depende de la combinación de los valores de handler y version especificados. En las aplicaciones administradas, las propiedades admitidas son `config`, `basics`, `steps` y `outputs`. Utilice `config` solo cuando necesite invalidar el comportamiento predeterminado del paso `basics`. Las propiedades basics y steps contienen los [elementos](create-uidefinition-elements.md), como cuadros de texto y listas desplegables, que se mostrarán en Azure Portal. La propiedad outputs se utiliza para asignar los valores de salida de los elementos especificados a los parámetros de la plantilla de Azure Resource Manager.

Se recomienda incluir `$schema`, aunque es opcional. Si se especifica, el valor de `version` debe coincidir con la versión en el identificador URI de `$schema`.

Puede usar un editor de JSON para crear el valor createUiDefinition y luego probarlo en el [espacio aislado createUiDefinition](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade) para obtener una vista previa. Para obtener más información sobre el espacio aislado, consulte [Probar la interfaz del portal de Azure Managed Applications](test-createuidefinition.md).

## <a name="config"></a>Config

La propiedad `config` es opcional. Utilícela para invalidar el comportamiento predeterminado del paso básico o para establecer la interfaz como un asistente paso a paso. Si se usa `config`, es la primera propiedad de la sección `parameters` del archivo **createUiDefinition.json**. En el ejemplo siguiente se muestran las propiedades disponibles.

```json
"config": {
    "isWizard": false,
    "basics": {
        "description": "Customized description with **markdown**, see [more](https://www.microsoft.com).",
        "subscription": {
            "constraints": {
                "validations": [
                    {
                        "isValid": "[expression for checking]",
                        "message": "Please select a valid subscription."
                    },
                    {
                        "permission": "<Resource Provider>/<Action>",
                        "message": "Must have correct permission to complete this step."
                    }
                ]
            },
            "resourceProviders": [
                "<Resource Provider>"
            ]
        },
        "resourceGroup": {
            "constraints": {
                "validations": [
                    {
                        "isValid": "[expression for checking]",
                        "message": "Please select a valid resource group."
                    }
                ]
            },
            "allowExisting": true
        },
        "location": {
            "label": "Custom label for location",
            "toolTip": "provide a useful tooltip",
            "resourceTypes": [
                "Microsoft.Compute/virtualMachines"
            ],
            "allowedValues": [
                "eastus",
                "westus2"
            ],
            "visible": true
        }
    }
},
```

### <a name="wizard"></a>Asistente

La propiedad `isWizard` le permite solicitar la validación correcta de cada paso antes de continuar con el paso siguiente. Cuando no se especifica la propiedad `isWizard`, el valor predeterminado es **falso** y la validación paso a paso no es necesaria.

Cuando `isWizard` está habilitado, es decir, se ha establecido en **true**, la pestaña **Aspectos básicos** está disponible y las demás pestañas están deshabilitadas. Cuando se selecciona el botón **Siguiente**, el icono de la pestaña indica si una pestaña se ha validado correctamente o no. Una vez que se han rellenado y validado los campos obligatorios de una pestaña, el botón **Siguiente** permite desplazarse a la pestaña siguiente. Cuando todas las pestañas se han validado correctamente, puede ir a la página **Revisar y crear** y seleccionar el botón **Crear** para comenzar la implementación.

:::image type="content" source="./media/create-uidefinition-overview/tab-wizard.png" alt-text="Asistente para pestañas":::

### <a name="override-basics"></a>Aspectos básicos de invalidación

La configuración básica permite personalizar el paso básico.

En `description`, proporcione una cadena habilitada para Markdown que describa el recurso. Se admiten vínculos y formato de varias líneas.

Los elementos `subscription` y `resourceGroup` permiten especificar validaciones adicionales. La sintaxis para especificar validaciones es idéntica a la de la validación personalizada de [cuadro de texto](microsoft-common-textbox.md). También puede especificar validaciones de `permission` en la suscripción o el grupo de recursos.  

El control de suscripción acepta una lista de espacios de nombres del proveedor de recursos. Por ejemplo, puede especificar **Microsoft.Compute**. En este caso, se muestra un mensaje de error cuando el usuario selecciona una suscripción que no admite el proveedor de recursos. El error se produce cuando el proveedor de recursos no está registrado en esa suscripción y el usuario no tiene permiso para registrarlo.  

El control de grupo de recursos tiene una opción `allowExisting`. Cuando es `true`, los usuarios pueden seleccionar grupos de recursos que ya tengan recursos. Esta marca se aplica principalmente a las plantillas de soluciones, donde el comportamiento predeterminado obliga a los usuarios a seleccionar un grupo de recursos nuevo o vacío. En la mayoría del resto de los escenarios, no es necesario especificar esta propiedad.  

En `location`, especifique las propiedades del control de ubicación que quiere invalidar. Cualquier propiedad que no se haya invalidado se establece en sus valores predeterminados. `resourceTypes` acepta una matriz de cadenas que contienen los nombres de tipo de recurso completos. Las opciones de ubicación están restringidas a solo las regiones que admiten los tipos de recursos.  `allowedValues`  acepta una matriz de cadenas de región. Solo esas regiones aparecen en la lista desplegable. Puede establecer tanto `allowedValues` como `resourceTypes`. El resultado es la intersección de ambas listas. Por último, la propiedad `visible` se puede utilizar para deshabilitar la lista desplegable de ubicación de forma condicional o por completo.  

## <a name="basics"></a>Aspectos básicos

El paso **Aspectos básicos** es el primer paso que se genera cuando Azure Portal analiza el archivo. De forma predeterminada, este paso permite a los usuarios elegir la suscripción, el grupo de recursos y la ubicación de la implementación.

:::image type="content" source="./media/create-uidefinition-overview/basics.png" alt-text="Valor predeterminado de Aspectos básicos":::

Puede agregar más elementos en esta sección. Cuando sea posible, agregue elementos que consulten los parámetros de toda la implementación, como el nombre de un clúster o las credenciales del administrador.

En el ejemplo siguiente se muestra un cuadro de texto que se ha agregado a los elementos predeterminados.

```json
"basics": [
    {
        "name": "textBox1",
        "type": "Microsoft.Common.TextBox",
        "label": "Textbox on basics",
        "defaultValue": "my text value",
        "toolTip": "",
        "visible": true
    }
]
```

## <a name="steps"></a>Pasos

La propiedad steps contiene cero o más pasos adicionales que se muestran después de los aspectos básicos. Cada paso contiene uno o varios elementos. Considere la posibilidad de agregar pasos por rol o nivel de aplicación que se está implementando. Por ejemplo, agregue un paso para las entradas de los nodos principales y un paso para los nodos de trabajo de un clúster.

```json
"steps": [
    {
        "name": "demoConfig",
        "label": "Configuration settings",
        "elements": [
          ui-elements-needed-to-create-the-instance
        ]
    }
]
```

## <a name="outputs"></a>Salidas

Azure Portal usa la propiedad `outputs` para asignar elementos de `basics` y `steps` a los parámetros de la plantilla de implementación de Azure Resource Manager. Las claves de este diccionario son los nombres de los parámetros de plantilla, y los valores son propiedades de los objetos de salida de los elementos a los que se hace referencia.

Para definir el nombre del recurso de la aplicación administrada, debe incluir un valor llamado `applicationResourceName` en la propiedad outputs. Si no establece este valor, la aplicación asigna un GUID al nombre. Puede incluir un cuadro de texto en la interfaz de usuario que requiera un nombre de usuario.

```json
"outputs": {
    "vmName": "[steps('appSettings').vmName]",
    "trialOrProduction": "[steps('appSettings').trialOrProd]",
    "userName": "[steps('vmCredentials').adminUsername]",
    "pwd": "[steps('vmCredentials').vmPwd.password]",
    "applicationResourceName": "[steps('appSettings').vmName]"
}
```

## <a name="resource-types"></a>Tipos de recursos

Para limitar las ubicaciones disponibles a solo las que admiten los tipos de recursos que se van a implementar, proporcione una matriz de los tipos de recursos. Si se proporciona más de un tipo de recurso, solo se devuelven las ubicaciones que admiten todos los tipos de recursos. Esta propiedad es opcional.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Azure.CreateUIDef",
    "version": "0.1.2-preview",
    "parameters": {
        "resourceTypes": ["Microsoft.Compute/disks"],
        "basics": [
          ...
```  

## <a name="functions"></a>Functions

CreateUiDefinition proporciona [funciones](create-uidefinition-functions.md) para trabajar con entradas y salidas de los elementos, y características como los condicionales. La sintaxis y funcionalidad de estas funciones son similares a las de las funciones de las plantillas de Azure Resource Manager.

## <a name="next-steps"></a>Pasos siguientes

El propio archivo createUiDefinition.json tiene un esquema simple. Su profundidad real procede de todos los elementos y funciones que admite. Dichos elementos se describen con más detalle en:

- [Elementos](create-uidefinition-elements.md)
- [Funciones](create-uidefinition-functions.md)

Un esquema JSON actual para createUiDefinition está disponible aquí: `https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json`.

Para un archivo de interfaz de usuario de ejemplo, vea [createUiDefinition.json](https://github.com/Azure/azure-managedapp-samples/blob/master/Managed%20Application%20Sample%20Packages/201-managed-app-using-existing-vnet/createUiDefinition.json).
