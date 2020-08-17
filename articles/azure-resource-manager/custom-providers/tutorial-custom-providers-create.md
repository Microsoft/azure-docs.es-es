---
title: Creación y uso de un proveedor personalizado
description: En este tutorial se explica cómo crear y utilizar un proveedor personalizado de Azure. Con los proveedores personalizados puede cambiar los flujos de trabajo en Azure.
author: jjbfour
ms.topic: tutorial
ms.date: 06/19/2019
ms.author: jobreen
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4f425af7681b666b42fbcc70ac0e4c31d9df6d49
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503759"
---
# <a name="create-and-use-a-custom-provider"></a>Creación y uso de un proveedor personalizado

Un proveedor personalizado es un contrato entre Azure y un punto de conexión. Con los proveedores personalizados, puede cambiar los flujos de trabajo en Azure. En este tutorial se muestra el proceso de creación de un proveedor personalizado. Si no está familiarizado con los proveedores personalizados de Azure, consulte [la introducción a los proveedores de recursos personalizados de Azure](overview.md).

## <a name="create-a-custom-provider"></a>Creación de un proveedor personalizado

> [!NOTE]
> En este tutorial no se muestra cómo crear un punto de conexión. Si no tiene ningún punto de conexión de RESTful, siga el [tutorial sobre la creación de puntos de conexión RESTful](./tutorial-custom-providers-function-authoring.md), que es la base del tutorial actual.

Después de crear un punto de conexión, puede crear un proveedor personalizado para generar un contrato entre el proveedor y el punto de conexión. Con un proveedor personalizado puede especificar una lista de definiciones de puntos de conexión:

```JSON
{
  "name": "myEndpointDefinition",
  "routingType": "Proxy",
  "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
}
```

Propiedad | Obligatorio | Descripción
---|---|---
**name** | Sí | El nombre de la definición del punto de conexión. Azure expone este nombre a través de su API en /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders<br>/resourceProviders/{resourceProviderName}/{endpointDefinitionName}
**routingType** | No | El tipo de contrato del punto de conexión. Si no se especifica el valor, el valor predeterminado es "Proxy".
**endpoint** | Sí | El punto de conexión al que enrutar las solicitudes. Este punto de conexión controla la respuesta, así como los efectos secundarios de la solicitud.

El valor del **punto de conexión** es la dirección URL del desencadenador de la aplicación de funciones de Azure. Los marcadores de posición `<yourapp>`, `<funcname>` y `<functionkey>` deben reemplazarse por los valores de la aplicación de funciones creada.

## <a name="define-custom-actions-and-resources"></a>Definición de acciones y recursos personalizados

El proveedor personalizado contiene una lista de definiciones de punto de conexión que se modelan con las propiedades **actions** y **resourceTypes**. La propiedad **actions** se asigna a las acciones personalizadas expuestas por el proveedor personalizado y la propiedad **resourceTypes** son los recursos personalizados. En este tutorial, el proveedor personalizado tiene una propiedad **actions** denominada `myCustomAction` y una propiedad **resourceTypes** denominada `myCustomResources`.

```JSON
{
  "properties": {
    "actions": [
      {
        "name": "myCustomAction",
        "routingType": "Proxy",
        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
      }
    ],
    "resourceTypes": [
      {
        "name": "myCustomResources",
        "routingType": "Proxy",
        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
      }
    ]
  },
  "location": "eastus"
}
```

## <a name="deploy-the-custom-provider"></a>Implementación del proveedor personalizado

> [!NOTE]
> Debe reemplazar los valores de **endpoint** por la dirección URL del desencadenador de la aplicación de funciones creada en el tutorial anterior.

Puede implementar el proveedor personalizado anterior mediante una plantilla de Azure Resource Manager:

```JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.CustomProviders/resourceProviders",
            "name": "myCustomProvider",
            "apiVersion": "2018-09-01-preview",
            "location": "eastus",
            "properties": {
                "actions": [
                    {
                        "name": "myCustomAction",
                        "routingType": "Proxy",
                        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
                    }
                ],
                "resourceTypes": [
                    {
                        "name": "myCustomResources",
                        "routingType": "Proxy",
                        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
                    }
                ]
            }
        }
    ]
}
```

## <a name="use-custom-actions-and-resources"></a>Uso de acciones y recursos personalizados

Después de crear un proveedor personalizado, puede usar las nuevas API de Azure. En las siguientes pestañas se explica cómo llamar a un proveedor personalizado y utilizarlo.

### <a name="custom-actions"></a>Acciones personalizadas

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

> [!NOTE]
> Debe reemplazar los marcadores de posición `{subscriptionId}` y `{resourceGroupName}` por la suscripción y el grupo de recursos donde implementó el proveedor personalizado.

```azurecli-interactive
az resource invoke-action --action myCustomAction \
                          --ids /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider \
                          --request-body
                            '{
                                "hello": "world"
                            }'
```

Parámetro | Obligatorio | Descripción
---|---|---
*action* | Sí | El nombre de la acción definida en el proveedor personalizado.
*ids* | Sí | El identificador de recurso del recurso personalizado.
*request-body* | No | El cuerpo de la solicitud que se enviará al punto de conexión.

# <a name="template"></a>[Plantilla](#tab/template)

Ninguno.

---

### <a name="custom-resources"></a>Recursos personalizados

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

> [!NOTE]
> Debe reemplazar los marcadores de posición `{subscriptionId}` y `{resourceGroupName}` por la suscripción y el grupo de recursos donde implementó el proveedor personalizado.

#### <a name="create-a-custom-resource"></a>Creación de un recurso personalizado

```azurecli-interactive
az resource create --is-full-object \
                   --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1 \
                   --properties
                    '{
                        "location": "eastus",
                        "properties": {
                            "hello" : "world"
                        }
                    }'
```

Parámetro | Obligatorio | Descripción
---|---|---
*is-full-object* | Sí | Indica que el objeto properties incluye otras opciones, como ubicación, etiquetas, SKU o plan.
*id* | Sí | Id. de recurso del recurso personalizado. Este identificador es una extensión del identificador de recurso del proveedor personalizado.
*properties* | Sí | El cuerpo de la solicitud que se enviará al punto de conexión.

#### <a name="delete-a-custom-resource"></a>Eliminación de un recurso personalizado

```azurecli-interactive
az resource delete --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1
```

Parámetro | Obligatorio | Descripción
---|---|---
*id* | Sí | Id. de recurso del recurso personalizado. Este identificador es una extensión del identificador de recurso del proveedor personalizado.

#### <a name="retrieve-a-custom-resource"></a>Recuperación de un recurso personalizado

```azurecli-interactive
az resource show --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1
```

Parámetro | Obligatorio | Descripción
---|---|---
*id* | Sí | Id. de recurso del recurso personalizado. Este identificador es una extensión del identificador de recurso del proveedor personalizado.

# <a name="template"></a>[Plantilla](#tab/template)

Una plantilla de Resource Manager de ejemplo:

```JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
            "name": "myCustomProvider/myTestResourceName1",
            "apiVersion": "2018-09-01-preview",
            "location": "eastus",
            "properties": {
                "hello": "world"
            }
        }
    ]
}
```

Parámetro | Obligatorio | Descripción
---|---|---
*resourceTypeName* | Sí | El valor `name` de la propiedad **resourceTypes** definido en el proveedor personalizado.
*resourceProviderName* | Sí | El nombre de instancia del proveedor personalizado.
*customResourceName* | Sí | Nombre del recurso personalizado.

---

> [!NOTE]
> Cuando haya terminado de implementar y usar el proveedor personalizado, recuerde limpiar todos los recursos creados, incluida la aplicación de funciones de Azure.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre los proveedores personalizados. Para más información, consulte:

- [Cómo: Adición de acciones personalizadas a la API REST de Azure](./custom-providers-action-endpoint-how-to.md)
- [Cómo: Adición de recursos personalizados a la API REST de Azure](./custom-providers-resources-endpoint-how-to.md)
