---
title: Administración de IoT Central desde la CLI de Azure | Microsoft Docs
description: En este artículo se describe cómo crear y administrar aplicaciones de IoT Central con la CLI. Puede ver, modificar y quitar la aplicación mediante la CLI.
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 03/27/2020
ms.topic: how-to
ms.custom: devx-track-azurecli
manager: philmea
ms.openlocfilehash: bd87f15ff63edf1da447faf986cad2f9591610dd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502977"
---
# <a name="manage-iot-central-from-azure-cli"></a>Administración de IoT Central desde la CLI de Azure

[!INCLUDE [iot-central-selector-manage](../../../includes/iot-central-selector-manage.md)]

En lugar de crear y administrar aplicaciones de IoT Central desde el sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral), puede usar la [CLI de Azure](/cli/azure/) para administrar las aplicaciones.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si prefiere ejecutar la CLI de Azure en el equipo local, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Cuando ejecute la CLI de Azure localmente, use el comando **az login** para iniciar sesión en Azure antes de probar los comandos descritos en este artículo.

> [!TIP]
> Si tiene que ejecutar los comandos de la CLI en otra suscripción a Azure, consulte [Cambio de la suscripción activa](/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest#change-the-active-subscription).

## <a name="install-the-extension"></a>Instalación de la extensión

Los comandos de este artículo forman parte de la extensión **azure-iot** de la CLI. Para instalar la extensión, ejecute el comando siguiente:

```azurecli-interactive
az extension add --name azure-iot
```

## <a name="create-an-application"></a>Crear una aplicación

Use el comando [az iot central app create](/cli/azure/iot/central/app?view=azure-cli-latest#az-iot-central-app-create) para crear una aplicación de IoT Central en su suscripción de Azure. Por ejemplo:

```azurecli-interactive
# Create a resource group for the IoT Central application
az group create --location "East US" \
    --name "MyIoTCentralResourceGroup"
```

```azurecli-interactive
# Create an IoT Central application
az iot central app create \
  --resource-group "MyIoTCentralResourceGroup" \
  --name "myiotcentralapp" --subdomain "mysubdomain" \
  --sku ST1 --template "iotc-pnp-preview" \
  --display-name "My Custom Display Name"
```

Estos comandos crean en primer lugar un grupo de recursos en la región Este de EE. UU. para la aplicación. En la siguiente tabla se describen los parámetros utilizados con el comando **az iot central app create**:

| Parámetro         | Descripción |
| ----------------- | ----------- |
| resource-group    | Grupo de recursos que contiene a la aplicación. Este grupo de recursos ya debe existir en la suscripción. |
| ubicación          | Este comando usa de forma predeterminada la ubicación del grupo de recursos. Actualmente, se puede crear una aplicación de IoT Central en las zonas geográficas **Australia**, **Asia Pacífico**, **Europa**, **Estados Unidos**, **Reino Unido** y **Japón**. |
| name              | Nombre de la aplicación en Azure Portal. |
| subdomain         | Subdominio en la dirección URL de la aplicación. En el ejemplo, la dirección URL de la aplicación es `https://mysubdomain.azureiotcentral.com`. |
| sku               | Actualmente, puede usar **ST1** o **ST2**. Consulte [Precios de Azure IoT Central](https://azure.microsoft.com/pricing/details/iot-central/). |
| template          | Plantilla de aplicación que se va a usar. Para más información, vea la tabla siguiente. |
| display-name      | Nombre de la aplicación tal como se muestra en la interfaz de usuario. |

[!INCLUDE [iot-central-template-list](../../../includes/iot-central-template-list.md)]

## <a name="view-your-applications"></a>Visualización de sus aplicaciones

Use el comando [az iot central app list](/cli/azure/iot/central/app?view=azure-cli-latest#az-iot-central-app-list) para enumerar las aplicaciones de IoT Central y ver los metadatos.

## <a name="modify-an-application"></a>Modificación de una aplicación

Use el comando [az iot central app update](/cli/azure/iot/central/app?view=azure-cli-latest#az-iot-central-app-update) para actualizar los metadatos de una aplicación de IoT Central. Por ejemplo, para cambiar el nombre para mostrar de la aplicación, use:

```azurecli-interactive
az iot central app update --name myiotcentralapp \
  --resource-group MyIoTCentralResourceGroup \
  --set displayName="My new display name"
```

## <a name="remove-an-application"></a>Eliminación de una aplicación

Use el comando [az iot central app delete](/cli/azure/iot/central/app?view=azure-cli-latest#az-iot-central-app-delete) para eliminar una aplicación de IoT Central. Por ejemplo:

```azurecli-interactive
az iot central app delete --name myiotcentralapp \
  --resource-group MyIoTCentralResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar aplicaciones de Azure IoT Central desde la CLI de Azure, le sugerimos el paso siguiente:

> [!div class="nextstepaction"]
> [Administrar su aplicación](howto-administer.md)
