---
title: Elemento de la interfaz de usuario ArmApiControl
description: Describe el elemento de la interfaz de usuario Microsoft.Solutions.ArmApiControl para Azure Portal. Se usa para las operaciones de llamadas API.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: tomfitz
ms.openlocfilehash: bbe36e072d10b81c421331b2212d8b161afd2693
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095205"
---
# <a name="microsoftcommonarmapicontrol-ui-element"></a>Elemento de la interfaz de usuario Microsoft.Common.ArmApiControl

ArmApiControl permite obtener resultados de una operación de la API de Azure Resource Manager. Use los resultados para rellenar el contenido dinámico en otros controles.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

No hay ninguna interfaz de usuario para este control.

## <a name="schema"></a>Schema

En el ejemplo siguiente, se muestra el esquema de este control:

```json
{
    "name": "testApi",
    "type": "Microsoft.Solutions.ArmApiControl",
    "request": {
        "method": "{HTTP-method}",
        "path": "{path-for-the-URL}",
        "body": {
            "key1": "val1",
            "key2": "val2"
        }
    }
}
```

## <a name="sample-output"></a>Salida de ejemplo

La salida del control no se muestra al usuario. En su lugar, el resultado de la operación se usa en otros controles.

## <a name="remarks"></a>Observaciones

- La propiedad `request.method` especifica el método HTTP. Solo se permiten los tipos `GET` o `POST`.
- La propiedad `request.path` especifica la ruta de acceso relativa de la dirección URL. Puede ser una ruta de acceso estática o se puede crear dinámicamente mediante la referencia a los valores de salida de los demás controles.
- La propiedad `request.body` es opcional. Úsela para especificar un cuerpo JSON que se enviará con la solicitud. Puede ser un cuerpo estático o se puede crear dinámicamente si se hace referencia a los valores de salida de otros controles.

## <a name="example"></a>Ejemplo

En el ejemplo siguiente, el elemento `providersApi` llama a una API para obtener una matriz de objetos de proveedor.

La propiedad `allowedValues` del elemento `providersDropDown` está configurada para obtener los nombres de los proveedores. Se muestran en la lista desplegable.

```json
{
    "name": "providersApi",
    "type": "Microsoft.Solutions.ArmApiControl",
    "request": {
        "method": "GET",
        "path": "[concat(subscription().id, '/providers/Microsoft.Network/expressRouteServiceProviders?api-version=2019-02-01')]"
    }
},
{
    "name": "providerDropDown",
    "type": "Microsoft.Common.DropDown",
    "label": "Provider",
    "toolTip": "The provider that offers the express route connection.",
    "constraints": {
        "allowedValues": "[map(steps('settings').providersApi.value, (item) => parse(concat('{\"label\":\"', item.name, '\",\"value\":\"', item.name, '\"}')))]",
        "required": true
    },
    "visible": true
}
```

Para ver un ejemplo del uso de ArmApiControl para comprobar la disponibilidad de un nombre de recurso, consulte [Microsoft.Common.TextBox](microsoft-common-textbox.md).

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
