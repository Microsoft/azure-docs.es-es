---
author: baanders
description: 'archivo de inclusión de Azure Digital Twins: configuración de la extensión de IoT más reciente'
ms.service: digital-twins
ms.topic: include
ms.date: 7/31/2020
ms.author: baanders
ms.openlocfilehash: 984739a728f6ac5e28eeb561e0d7b6ec0485ca13
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496872"
---
En primer lugar, ejecute este comando para ver una lista de todas las extensiones que ya tenga instaladas.

```azurecli-interactive
az extension list
```

La salida es una matriz de todas las extensiones que tiene actualmente. Busque el campo `"name"` para cada entrada de la lista con el fin de ver los nombres de las extensiones.

Use la salida para determinar cuál de los siguientes comandos se ejecutará para la instalación de la extensión (puede ejecutar más de uno).
* Si la lista contiene `azure-iot`: Ya tiene la extensión. Ejecute este comando para asegurarse de que tiene la actualización más reciente y de que no hay otras actualizaciones disponibles:

   ```azurecli-interactive
   az extension update --name azure-iot
   ```

* Si la lista **no** contiene `azure-iot`: Debe instalar la extensión. Use este comando:

    ```azurecli-interactive
    az extension add --name azure-iot
    ```

* Si la lista contiene `azure-iot-cli-ext`: Se trata de la versión heredada de la extensión. Solo se debe instalar una versión de la extensión a la vez, por lo que debe desinstalar la heredada. Use este comando:

   ```azurecli-interactive
   az extension remove --name azure-cli-iot-ext
   ```