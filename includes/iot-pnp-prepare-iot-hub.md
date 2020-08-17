---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 03/17/2020
ms.custom: references_regions
ms.openlocfilehash: adc621f932462422202d9f16fd539f5ecc7c3d8e
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87336910"
---
## <a name="prepare-an-iot-hub"></a>Preparación de un centro de IoT

Necesitará una instancia de Azure IoT Hub en la suscripción de Azure para completar los pasos de este artículo. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Si utiliza la CLI de Azure de forma local, inicie sesión primero en la suscripción de Azure mediante `az login`. Si está ejecutando estos comandos en Azure Cloud Shell, su sesión se iniciará automáticamente.

Si usa la CLI de Azure localmente, la versión de `az` debe ser la **2.8.0** o posterior. Azure Cloud Shell usa la versión más reciente. Use el comando `az --version` para comprobar la versión instalada en la máquina.

Ejecute el siguiente comando para agregar la extensión de IoT de Microsoft Azure para la CLI de Azure a la instancia:

```azurecli-interactive
az extension add --name azure-iot
```

Si aún no tiene un centro de IoT para usar, ejecute los siguientes comandos para crear un grupo de recursos y un centro de IoT de nivel gratuito en su suscripción. Reemplace `<YourIoTHubName>` por el nombre del centro de conectividad que prefiera:

```azurecli-interactive
az group create --name my-pnp-resourcegroup \
    --location centralus
az iot hub create --name <YourIoTHubName> \
    --resource-group my-pnp-resourcegroup --sku F1
```

> [!NOTE]
> IoT Plug and Play se encuentra actualmente disponible en los centros de IoT creados en las regiones Centro de EE. UU., Norte de Europa y Japón Oriental. La compatibilidad con IoT Plug and Play no se incluye en los centros de IoT de nivel básico.

Ejecute el siguiente comando la nueva identidad del dispositivo en su centro de IoT. Reemplace los marcadores de posición `<YourIoTHubName>` y `<YourDeviceID>` por su propio _nombre de centro de IoT_ y un _identificador de dispositivo_ de su elección.

```azurecli-interactive
az iot hub device-identity create --hub-name <YourIoTHubName> --device-id <YourDeviceID>
```
