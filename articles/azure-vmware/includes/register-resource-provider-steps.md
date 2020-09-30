---
title: Registro del proveedor de recursos de Azure VMware Solution
description: Pasos para registrar el proveedor de recursos de Azure VMware Solution.
ms.topic: include
ms.date: 09/04/2020
ms.openlocfilehash: 08741181b778b44289868a9f3fb1c2368955248c
ms.sourcegitcommit: 8791f69d44150767807d215cafc4076f3ed43f9f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89512387"
---
Para usar Azure VMware Solution, primero debe registrar el proveedor de recursos con la suscripción.

```
azurecli-interactive
az provider register -n Microsoft.AVS --subscription <your subscription ID>
```

Para ver otras formas de registrar el proveedor de recursos, consulte [Tipos y proveedores de recursos de Azure](../../azure-resource-manager/management/resource-providers-and-types.md).