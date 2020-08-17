---
title: 'Implementación de módulos desde una línea de comandos de la CLI de Azure: Azure IoT Edge'
description: Use la CLI de Azure con la extensión IoT de Azure para enviar un módulo de IoT Edge desde la instancia de IoT Hub al dispositivo IoT Edge, tal como se configura en un manifiesto de implementación.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/16/2019
ms.topic: conceptual
ms.reviewer: menchi
ms.service: iot-edge
ms.custom: devx-track-azurecli
services: iot-edge
ms.openlocfilehash: 222e3e75d61096dc7aebb409213b8016e478c72b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501583"
---
# <a name="deploy-azure-iot-edge-modules-with-azure-cli"></a>Implementación de módulos de Azure IoT Edge con la CLI de Azure

Una vez que ha creado módulos de IoT Edge con su lógica empresarial, querrá implementarlos en sus dispositivos para usarlos en el perímetro. Si tiene varios módulos que funcionan conjuntamente para recopilar y procesar datos, puede implementarlos todos a la vez y declarar las reglas de enrutamiento que los conectan.

La [CLI de Azure](https://docs.microsoft.com/cli/azure?view=azure-cli-latest) es una herramienta de la línea de comandos multiplataforma y de código abierto para la administración de recursos de Azure, como IoT Edge. Esta permite administrar recursos de Azure IoT Hub, instancias de servicio de aprovisionamiento de dispositivos y centros vinculados listos para usar. La extensión de IoT enriquece la CLI de Azure con características como administración de dispositivos y funcionalidad completa de IoT Edge.

En este artículo se muestra cómo crear un manifiesto de implementación de JSON y, luego, usar ese archivo para insertar la implementación en un dispositivo IoT Edge. Para información sobre la creación de una implementación dirigida a varios dispositivos en función de sus etiquetas compartidas, consulte [Implementación y supervisión de módulos de IoT Edge a escala](how-to-deploy-cli-at-scale.md).

## <a name="prerequisites"></a>Requisitos previos

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-using-cli.md) en la suscripción de Azure.
* Un [dispositivo de IoT Edge](how-to-register-device.md#register-with-the-azure-cli) que tenga instalado el entorno de ejecución de Azure IoT Edge.
* La [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) en su entorno. Como mínimo, la versión de la CLI de Azure debe ser la 2.0.70. Use `az --version` para asegurarse. Esta versión admite comandos az extension e introduce la plataforma de comandos de Knack.
* La [extensión de IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension).

## <a name="configure-a-deployment-manifest"></a>Configuración de un manifiesto de implementación

Un manifiesto de implementación es un documento JSON que describe qué módulos se van a implementar, cómo fluyen los datos entre los módulos y las propiedades deseadas de los módulos gemelos. Para más información sobre los manifiestos de implementación y cómo crearlos, consulte [Descripción de cómo se pueden utilizar, configurar y reutilizar los módulos de IoT Edge](module-composition.md).

Para implementar módulos con la CLI de Azure, guarde el manifiesto de implementación localmente como un archivo .json. Usará la ruta de acceso al archivo en la sección siguiente, al ejecutar el comando para aplicar la configuración al dispositivo.

Este es un manifiesto de implementación básico con un módulo como ejemplo:

```json
{
  "content": {
    "modulesContent": {
      "$edgeAgent": {
        "properties.desired": {
          "schemaVersion": "1.0",
          "runtime": {
            "type": "docker",
            "settings": {
              "minDockerVersion": "v1.25",
              "loggingOptions": "",
              "registryCredentials": {}
            }
          },
          "systemModules": {
            "edgeAgent": {
              "type": "docker",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
                "createOptions": "{}"
              }
            },
            "edgeHub": {
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
              }
            }
          },
          "modules": {
            "SimulatedTemperatureSensor": {
              "version": "1.0",
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
                "createOptions": "{}"
              }
            }
          }
        }
      },
      "$edgeHub": {
        "properties.desired": {
          "schemaVersion": "1.0",
          "routes": {
            "upstream": "FROM /messages/* INTO $upstream"
          },
          "storeAndForwardConfiguration": {
            "timeToLiveSecs": 7200
          }
        }
      },
      "SimulatedTemperatureSensor": {
        "properties.desired": {
          "SendData": true,
          "SendInterval": 5
        }
      }
    }
  }
}
```

## <a name="deploy-to-your-device"></a>Implementación en el dispositivo

Para implementar los módulos en el dispositivo, aplique el manifiesto de implementación que configuró con la información del módulo.

Cambie los directorios a la carpeta donde se guarda el manifiesto de implementación. Si ha usado una de las plantillas de IoT Edge para VS Code, use el archivo `deployment.json` de la carpeta **config** del directorio de la solución y no en el archivo `deployment.template.json`.

Utilice el siguiente comando para aplicar la configuración a un dispositivo IoT Edge:

   ```azurecli
   az iot edge set-modules --device-id [device id] --hub-name [hub name] --content [file path]
   ```

El parámetro del id. de dispositivo distingue mayúsculas y minúsculas. El parámetro "content" apunta al archivo del manifiesto de implementación que guardó.

   ![az iot edge set-modules output](./media/how-to-deploy-cli/set-modules.png)

## <a name="view-modules-on-your-device"></a>Visualización de módulos en el dispositivo

Una vez que los módulos se han implementado en el dispositivo, puede verlos todos con el siguiente comando:

Vea los módulos en el dispositivo de IoT Edge:

   ```azurecli
   az iot hub module-identity list --device-id [device id] --hub-name [hub name]
   ```

El parámetro del id. de dispositivo distingue mayúsculas y minúsculas.

   ![az iot hub module-identity list output](./media/how-to-deploy-cli/list-modules.png)

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [implementar y supervisar módulos de IoT Edge a escala](how-to-deploy-at-scale.md)
