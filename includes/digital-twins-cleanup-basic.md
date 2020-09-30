---
author: baanders
description: Archivo de inclusión para limpiar una instancia básica de Azure Digital Twins y el registro de la aplicación
ms.service: digital-twins
ms.topic: include
ms.date: 8/13/2020
ms.author: baanders
ms.openlocfilehash: e8f167555434338416381aaf8978a3d551686fb2
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90606916"
---
Cuando ya no necesite los recursos creados en este tutorial, siga estos pasos para eliminarlos.

Con [Azure Cloud Shell](https://shell.azure.com), puede eliminar todos los recursos de Azure de un grupo mediante el comando [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest&preserve-view=true#az-group-delete). Esta acción quita el grupo de recursos y la instancia de Azure Digital Twins.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. 

Abra una instancia de Azure Cloud Shell y ejecute el siguiente comando para eliminar el grupo de recursos y todo lo que contiene.

```azurecli
az group delete --name <your-resource-group>
```

A continuación, use este comando para eliminar el registro de aplicación de Azure Active Directory que creó para la aplicación cliente:

```azurecli
az ad app delete --id <your-application-ID>
```