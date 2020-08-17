---
title: archivo de inclusión
description: archivo de inclusión
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: include
ms.date: 08/17/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: b48e5e28b15d17796138d493a7d9c628791df2e6
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830142"
---
## <a name="enable-the-event-grid-resource-provider"></a>Habilitar el proveedor de recursos de Event Grid

Si aún no ha usado anteriormente Event Grid en su suscripción de Azure, puede que tenga que registrar el proveedor de recursos de Event Grid. Ejecute el siguiente comando para registrar el proveedor:

```azurecli-interactive
az provider register --namespace Microsoft.EventGrid
```

El registro puede tardar unos instantes en finalizar. Para comprobar el estado, ejecute:

```azurecli-interactive
az provider show --namespace Microsoft.EventGrid --query "registrationState"
```

Cuando `registrationState` sea `Registered`, estará preparado para continuar.