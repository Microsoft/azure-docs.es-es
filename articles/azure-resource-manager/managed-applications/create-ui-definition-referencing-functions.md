---
title: Creación de funciones de referencia para definiciones de interfaz de usuario
description: Describe las funciones que se van a usar al crear definiciones de interfaz de usuario para Azure Portal que hacen referencia a otros objetos.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: ad21c5b34a58c35b2cef5e430be7cb8cd1296402
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095209"
---
# <a name="createuidefinition-referencing-functions"></a>Funciones de referencia CreateUiDefinition

Se trata de las funciones que se van a usar para hacer referencia a salidas de las propiedades o el contexto de un archivo de CreateUiDefinition.

## <a name="basics"></a>basics

Devuelve los valores de salida de un elemento que se define en el paso [Aspectos básicos](create-uidefinition-overview.md#basics). Pase el nombre del elemento como un parámetro a esta función.

Para obtener los valores de salida de los elementos en otros pasos, utilice la función [steps()](#steps).

En el ejemplo siguiente, se devuelve el resultado del elemento denominado `clusterName` en el paso Basics:

```json
"[basics('clusterName')]"
```

Los valores devueltos varían en función del tipo de elemento que se recupera.

## <a name="location"></a>ubicación

Devuelve la ubicación seleccionada en el paso Basics o el contexto actual.

El ejemplo siguiente devuelve un valor como `"westus"`:

```json
"[location()]"
```

## <a name="resourcegroup"></a>resourceGroup

Devuelve detalles sobre la instancia de resourceGroup seleccionada en el paso Aspectos básicos o en el contexto actual.

En el ejemplo siguiente:

```json
"[resourceGroup()]"
```

Devuelve las siguientes propiedades:

```json
{
    "mode": "New" or "Existing",
    "name": "{resourceGroupName}",
    "location": "{resourceGroupLocation}"
}
```

Puede obtener cualquier valor concreto con la notación de puntos.

```json
"[resourceGroup().name]"
```

## <a name="steps"></a>steps

Devuelve los elementos de un paso especificado. Pase el nombre del paso como un parámetro a esta función. En los elementos devueltos, puede obtener valores concretos de propiedad.

Para obtener los valores de salida de los elementos en el paso Aspectos básicos, use la función [basics()](#basics).

El ejemplo siguiente devuelve el paso denominado `vmParameters`. En ese paso, es un elemento denominado `adminUsername`.

```json
"[steps('vmParameters').adminUsername]"
```

## <a name="subscription"></a>subscription

Devuelve propiedades para la suscripción seleccionada en el paso Aspectos básicos o el contexto actual.

En el ejemplo siguiente:

```json
"[subscription()]"
```

Devuelve las siguientes propiedades:

```json
{
    "id": "/subscriptions/{subscription-id}",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}",
    "displayName": "{name-of-subscription}"
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para una introducción para desarrollar la interfaz del portal, vea [CreateUiDefinition.json para la experiencia de creación de aplicaciones administradas de Azure](create-uidefinition-overview.md).
