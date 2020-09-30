---
title: Creación de un centro de IoT mediante la CLI de Azure | Microsoft Docs
description: Aprenda a usar los comandos de la CLI de Azure para crear un grupo de recursos y, luego, crear en él un centro de IoT. También aprenda a quitar el centro.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/23/2018
ms.author: robinsh
ms.openlocfilehash: 69372e4c212e2ce81bcd4c91d460aa191a1d3476
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90087871"
---
# <a name="create-an-iot-hub-using-the-azure-cli"></a>Creación de una instancia de IoT Hub mediante la CLI de Azure

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

En este artículo se muestra cómo utilizar la CLI de Azure para crear una instancia de IoT Hub.

## <a name="prerequisites"></a>Prerequisites

Para completar este procedimiento, se necesita una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-and-set-your-azure-account"></a>Inicio de sesión y configuración de la cuenta de Azure

Si está ejecutando la CLI de Azure localmente, en lugar de usar Cloud Shell, deberá iniciar sesión en su cuenta de Azure.

En el símbolo del sistema, ejecute el [comando de inicio de sesión](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli):

   ```azurecli
   az login
   ```

Siga las instrucciones para realizar la autenticación mediante el código e inicie sesión en la cuenta de Azure a través de un explorador web.

## <a name="create-an-iot-hub"></a>Creación de un IoT Hub

Use la CLI de Azure para crear un grupo de recursos y, luego, agregue una instancia de IoT Hub.

1. Cuando crea una instancia de IoT Hub, debe crearla en un grupo de recursos. Use un grupo de recursos existente o ejecute el [comando siguiente para crear un grupo de recursos](https://docs.microsoft.com/cli/azure/resource):
    
   ```azurecli-interactive
   az group create --name {your resource group name} --location westus
   ```

   > [!TIP]
   > El ejemplo anterior crea el grupo de recursos en la ubicación del oeste de EE. UU. Puede ejecutar el siguiente comando para ver una lista de las ubicaciones disponibles: 
   >
   > ```azurecli-interactive
   > az account list-locations -o table
   > ```
   >

2. Ejecute el [comando siguiente para crear una instancia de IoT Hub](https://docs.microsoft.com/cli/azure/iot/hub#az-iot-hub-create) en el grupo de recursos, con un nombre único global para la instancia de IoT Hub:
    
   ```azurecli-interactive
   az iot hub create --name {your iot hub name} \
      --resource-group {your resource group name} --sku S1
   ```

   [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]


El comando anterior crea una instancia de IoT Hub en el plan de tarifa S1 según el que factura. Para obtener más información, vea el artículo sobre [precios de IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub/).

## <a name="remove-an-iot-hub"></a>Eliminación de una instancia de IoT Hub

Puede usar la CLI de Azure para [eliminar un recurso individual](https://docs.microsoft.com/cli/azure/resource), como una instancia de IoT Hub, o para eliminar un grupo de recursos y todos sus recursos, incluida cualquier instancia de IoT Hub.

Ejecute el comando siguiente para [eliminar una instancia de IoT Hub](https://docs.microsoft.com/cli/azure/iot/hub#az-iot-hub-delete):

```azurecli-interactive
az iot hub delete --name {your iot hub name} -\
  -resource-group {your resource group name}
```

Ejecute el comando siguiente para [eliminar un grupo de recursos](https://docs.microsoft.com/cli/azure/group#az-group-delete) y todos sus recursos:

```azurecli-interactive
az group delete --name {your resource group name}
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo usar una instancia de IoT Hub, consulte los siguientes artículos:

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)
* [Uso de Azure Portal para administrar IoT Hub](iot-hub-create-through-portal.md)
