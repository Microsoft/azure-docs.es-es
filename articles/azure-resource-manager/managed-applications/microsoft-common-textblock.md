---
title: Elemento de la interfaz de usuario TextBlock
description: Describe el elemento de la interfaz de usuario Microsoft.Common.TextBlock para Azure Portal. Se usa para agregar texto a la interfaz.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: ba2c4d410891910c29ee1fda1065f8230ab171bf
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87063883"
---
# <a name="microsoftcommontextblock-ui-element"></a>Elemento de interfaz de usuario Microsoft.Common.TextBlock

Un control que se puede usar para agregar texto a la interfaz del portal.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

![Microsoft.Common.TextBox](./media/managed-application-elements/microsoft-common-textblock.png)

## <a name="schema"></a>Schema

```json
{
  "name": "text1",
  "type": "Microsoft.Common.TextBlock",
  "visible": true,
  "options": {
    "text": "Please provide the configuration values for your application.",
    "link": {
      "label": "Learn more",
      "uri": "https://www.microsoft.com"
    }
  }
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
"Please provide the configuration values for your application. Learn more"
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
