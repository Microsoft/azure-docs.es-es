---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 11/14/2019
ms.author: glenga
ms.openlocfilehash: b67e2bf2ae5af2feb334e898ce69fd5b959c7cf0
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689576"
---
```json
{
    "version": "2.0",
    "extensionBundle": {
        "id": "Microsoft.Azure.Functions.ExtensionBundle",
        "version": "[1.*, 2.0.0)"
    }
}
```

Las siguientes propiedades están disponibles en `extensionBundle`:

| Propiedad | Descripción |
| -------- | ----------- |
| id | Espacio de nombres de las agrupaciones de extensiones de Microsoft Azure Functions. |
| version | Versión de la agrupación que va a instalar. Functions Runtime siempre elige la máxima versión permitida definida por el rango o intervalo de versiones. El valor de versión anterior permite todas las versiones de agrupaciones desde la 1.0.0 en adelante pero sin incluir la 2.0.0. Para más información, consulte la [notación de intervalo para especificar intervalos de versiones](/nuget/reference/package-versioning#version-ranges). |