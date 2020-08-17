---
author: dominicbetts
ms.author: dominicbetts
ms.service: iot-pnp
ms.topic: include
ms.date: 11/15/2019
ms.openlocfilehash: 6cd983b6421fc821853aa22d4dc2c297f672d292
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87336909"
---
## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea leer artículos de IoT Plug and Play adicionales, puede conservar los recursos que usó en este artículo y reutilizarlos. De lo contrario, puede eliminar los recursos que creó para este artículo a fin de evitar cargos adicionales.

Puede eliminar el centro y el dispositivo registrado a la vez si elimina todo el grupo de recursos con el siguiente comando de la CLI de Azure. No use este comando si estos recursos comparten un grupo de recursos con otros recursos que desee conservar.

```azurecli-interactive
az group delete --name <YourResourceGroupName>
```

Para eliminar solo la instancia de IoT Hub, ejecute el siguiente comando con la CLI de Azure:

```azurecli-interactive
az iot hub delete --name <YourIoTHubName>
```

Para eliminar solo la identidad del dispositivo que registró en IoT Hub, ejecute el comando siguiente mediante la CLI de Azure:

```azurecli-interactive
az iot hub device-identity delete --hub-name <YourIoTHubName> --device-id <YourDeviceID>
```

También puede que quiera quitar los archivos de ejemplo clonados de la máquina de desarrollo.
