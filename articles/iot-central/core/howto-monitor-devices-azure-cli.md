---
title: Supervisar la conectividad de dispositivos con Azure IoT Central Explorer
description: Supervise los mensajes de los dispositivo y observe los cambios de los dispositivos gemelos mediante la CLI de IoT Central Explorer.
author: viv-liu
ms.author: viviali
ms.date: 03/27/2020
ms.topic: how-to
ms.service: iot-central
ms.custom: devx-track-azurecli, device-developer
services: iot-central
manager: corywink
ms.openlocfilehash: 276513e41b1595180acb0a596b236428032d87a6
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90015983"
---
# <a name="monitor-device-connectivity-using-azure-cli"></a>Supervisión de la conectividad de dispositivos mediante la CLI de Azure

*Este tema se aplica a desarrolladores de dispositivos y generadores de soluciones.*

Use la extensión de IoT de la CLI de Azure para ver los mensajes que sus dispositivos envían a IoT Central y observe los cambios en el dispositivo gemelo. Puede usar esta herramienta para depurar y observar la conectividad del dispositivo y diagnosticar los problemas de los mensajes del dispositivo que no llegan a la nube o los dispositivos que no responden a cambios gemelos.

[Consulte la referencia de extensiones de la CLI de Azure para obtener más detalles](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/central?view=azure-cli-latest)

## <a name="prerequisites"></a>Prerrequisitos

+ CLI de Azure instalada, versión 2.7.0 o posterior. Compruebe la versión de la CLI de Azure al ejecutar `az --version`. Obtenga información sobre la instalación y actualización en los [docs de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli)
+ Una cuenta profesional o educativa de Azure, agregada como usuario en una aplicación de IoT Central.

## <a name="install-the-iot-central-extension"></a>Instalación de la extensión de IoT Central

Ejecute el siguiente comando desde su línea de comando paras instalarlo:

```azurecli
az extension add --name azure-iot
```

Compruebe la versión de la extensión. Para ello, ejecute el siguiente comando:

```azurecli
az --version
```

Debe ver que la extensión azure-iot es 0.9.9 o posterior. Si no es así, ejecute:

```azurecli
az extension update --name azure-iot
```

## <a name="using-the-extension"></a>Uso de la extensión

En las secciones siguientes se describen comandos y opciones comunes que puede usar al ejecutar `az iot central`. Para ver el conjunto completo de comandos y opciones, pase `--help` a `az iot central` o a cualquiera de sus subcomandos.

### <a name="login"></a>Inicio de sesión

Empiece por iniciar sesión en la CLI de Azure. 

```azurecli
az login
```

### <a name="get-the-application-id-of-your-iot-central-app"></a>Obtener el id. de la aplicación de la aplicación de IoT Central
En **Administración/Configuración de la aplicación**, copie el **Id. de la aplicación**. Se usará este valor en pasos posteriores.

### <a name="monitor-messages"></a>Controlar mensajes
Supervise los mensajes que se envían a la aplicación de IoT Central desde sus dispositivos. La salida incluye todos los encabezados y anotaciones.

```azurecli
az iot central diagnostics monitor-events --app-id <app-id> --properties all
```

### <a name="view-device-properties"></a>Visualización de las propiedades del dispositivo
Consulte las propiedades actuales del dispositivo de lectura y de lectura y escritura para un dispositivo determinado.

```azurecli
az iot central device twin show --app-id <app-id> --device-id <device-id>
```

## <a name="next-steps"></a>Pasos siguientes

Si es desarrollador de dispositivos, el siguiente paso sugerido es leer información sobre la [conectividad de dispositivos en Azure IoT Central](./concepts-get-connected.md).
