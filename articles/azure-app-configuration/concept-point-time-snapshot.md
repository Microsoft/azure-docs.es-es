---
title: Recuperación de pares clave-valor desde un momento dado
titleSuffix: Azure App Configuration
description: Recupere pares clave-valor antiguos mediante instantáneas de un momento dado en Azure App Configuration, lo que mantendrá un registro de los cambios en los pares clave-valor.
services: azure-app-configuration
author: lisaguthrie
ms.author: lcozzens
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: b1d559d82cb22d8a787785c6d8c6a5101d89793a
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88586637"
---
# <a name="point-in-time-snapshot"></a>Instantánea en un momento dado

Azure App Configuration mantiene un registro de los cambios realizados en los pares clave-valor. Este registro proporciona una escala de tiempo de los cambios de clave-valor. Puede reconstruir el historial de cualquier par clave-valor y proporcionar su valor anterior en cualquier momento dentro del período del historial de claves (7 días para los almacenes de nivel gratis o 30 días para los almacenes de nivel estándar). Con el uso de esta característica puede "viajar en el tiempo" al pasado y recuperar un antiguo par clave-valor. Por ejemplo, puede recuperar los valores de configuración utilizados antes de la implementación más reciente, para revertir la aplicación a la configuración anterior.

## <a name="key-value-retrieval"></a>Recuperación de pares clave-valor

Puede usar Azure Portal o la CLI para recuperar los pares de clave-valor anteriores. En la CLI de Azure, use `az appconfig revision list`, y agregue los parámetros adecuados para recuperar los valores necesarios.  Especifique la instancia de Azure App Configuration proporcionando el nombre del almacén (`--name <app-config-store-name>`) o mediante una cadena de conexión (`--connection-string <your-connection-string>`). Restrinja la salida especificando un punto concreto en el tiempo (`--datetime`) y el número máximo de elementos que se van a devolver (`--top`).

Si no tiene la CLI de Azure instalada localmente, también puede usar Azure Cloud Shell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Recupere todos los cambios registrados en los pares clave-valor.

```azurecli-interactive
az appconfig revision list --name <your-app-config-store-name>.
```

Recupere todos los cambios registrados para la clave `environment` y las etiquetas `test` y `prod`.

```azurecli-interactive
az appconfig revision list --name <your-app-config-store-name> --key environment --label test,prod
```

Recupere todos los cambios registrados en el espacio de clave jerárquico `environment:prod`.

```azurecli-interactive
az appconfig revision list --name <your-app-config-store-name> --key environment:prod:* 
```

Recupere todos los cambios registrados para la clave `color` en un momento dado específico.

```azurecli-interactive
az appconfig revision list --connection-string <your-app-config-connection-string> --key color --datetime "2019-05-01T11:24:12Z" 
```

Recupere los últimos 10 cambios registrados en los pares clave-valor y devuelva solo los valores de `key`, `label`y la marca de tiempo `last_modified`.

```azurecli-interactive
az appconfig revision list --name <your-app-config-store-name> --top 10 --fields key label last_modified
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cree una aplicación web ASP.NET Core](./quickstart-aspnet-core-app.md)  
