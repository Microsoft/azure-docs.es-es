---
title: Cómo implementar el módulo OPC Twin para Azure desde cero | Microsoft Docs
description: En este artículo se explica cómo implementar OPC Twin desde cero con la hoja IoT Edge de Azure Portal y también mediante la CLI de AZ.
author: dominicbetts
ms.author: dobett
ms.date: 11/26/2018
ms.topic: conceptual
ms.service: industrial-iot
ms.custom: devx-track-azurecli
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: a0c5c601b0d3bc0d862ea4984ee2c6d4b76d13ed
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502467"
---
# <a name="deploy-opc-twin-module-and-dependencies-from-scratch"></a>Implementación del módulo y las dependencias de OPC Twin desde cero

El módulo OPC Twin se ejecuta en IoT Edge y proporciona varios servicios de Edge para los servicios de registro y dispositivo gemelo OPC. 

Hay varias opciones para implementar módulos en la puerta de enlace de [Azure IoT Edge](https://azure.microsoft.com/services/iot-edge/), entre ellas

- [Implementación de la hoja de IoT Edge desde Azure Portal](https://docs.microsoft.com/azure/iot-edge/how-to-deploy-modules-portal)
- [Implementación mediante la CLI de AZ](https://docs.microsoft.com/azure/iot-edge/how-to-deploy-monitor-cli)

> [!NOTE]
> Para obtener más información sobre las instrucciones y los detalles de la implementación, vea el [repositorio](https://github.com/Azure/azure-iiot-components) de GitHub.

## <a name="deployment-manifest"></a>Manifiesto de implementación

Todos los módulos se implementan mediante un manifiesto de implementación.  A continuación, se muestra un manifiesto de ejemplo para implementar [OPC Publisher](https://github.com/Azure/iot-edge-opc-publisher) y [OPC Twin](https://github.com/Azure/azure-iiot-opc-twin-module).

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
                "createOptions": ""
              }
            },
            "edgeHub": {
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}], \"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
              }
            }
          },
          "modules": {
            "opctwin": {
              "version": "1.0",
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/iotedge/opc-twin:latest",
                "createOptions": "{\"NetworkingConfig\": {\"EndpointsConfig\": {\"host\": {}}}, \"HostConfig\": {\"NetworkMode\": \"host\" }}"
              }
            },
            "opcpublisher": {
              "version": "2.0",
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/iotedge/opc-publisher:latest",
                "createOptions": "{\"Hostname\":\"publisher\",\"Cmd\":[\"publisher\",\"--pf=./pn.json\",\"--di=60\",\"--to\",\"--aa\",\"--si=0\",\"--ms=0\"],\"ExposedPorts\":{\"62222/tcp\":{}},\"NetworkingConfig\":{\"EndpointsConfig\":{\"host\":{}}},\"HostConfig\":{\"NetworkMode\":\"host\",\"PortBindings\":{\"62222/tcp\":[{\"HostPort\":\"62222\"}]}}}"
              }
            }
          }
        }
      },
      "$edgeHub": {
        "properties.desired": {
          "schemaVersion": "1.0",
          "routes": {
            "opctwinToIoTHub": "FROM /messages/modules/opctwin/* INTO $upstream",
            "opcpublisherToIoTHub": "FROM /messages/modules/opcpublisher/* INTO $upstream"
          },
          "storeAndForwardConfiguration": {
            "timeToLiveSecs": 7200
          }
        }
      }
    }
  }
}
```

## <a name="deploying-from-azure-portal"></a>Implementación desde Azure Portal

La manera más fácil de implementar los módulos en un dispositivo de puerta de enlace de Azure IoT Edge es a través de Azure Portal.  

### <a name="prerequisites"></a>Prerequisites

1. Implementación de las [dependencias](howto-opc-twin-deploy-dependencies.md) de OPC Twin y obtención del archivo `.env` resultante. Anote la instancia de `hub name` implementada en la variable `PCS_IOTHUBREACT_HUB_NAME` del archivo `.env` resultante.

2. Registre e inicie un puerta de enlace de IoT Edge de [Linux](https://docs.microsoft.com/azure/iot-edge/how-to-install-iot-edge-linux) o [Windows](https://docs.microsoft.com/azure/iot-edge/how-to-install-iot-edge-windows) y anote su `device id`.

### <a name="deploy-to-an-edge-device"></a>Implementación en un dispositivo de Edge

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya a IoT Hub.

2. Seleccione **IoT Edge** en el menú izquierdo.

3. Haga clic en el identificador del dispositivo de destino en la lista de dispositivos.

4. Seleccione **Set modules** (Establecer módulos).

5. En la sección **Módulos de implementación** de la página, seleccione **Agregar** y **Módulo IoT Edge**.

6. En el diálogo **Módulo personalizado de IoT Edge** use `opctwin` como nombre del módulo y, después, especifique el *URI de la imagen* del contenedor como

   ```bash
   mcr.microsoft.com/iotedge/opc-twin:latest
   ```

   Como *Opciones de creación del contenedor*, use el siguiente JSON:

   ```json
   {"NetworkingConfig": {"EndpointsConfig": {"host": {}}}, "HostConfig": {"NetworkMode": "host" }}
   ```

   Rellene los campos opcionales si es necesario. Para más información sobre las opciones de creación del contenedor, la directiva de reinicio y el estado deseado, consulte [Propiedades deseadas de EdgeAgent](https://docs.microsoft.com/azure/iot-edge/module-edgeagent-edgehub#edgeagent-desired-properties). Para más información sobre el módulo gemelo, consulte [Definición o actualización de las propiedades deseadas](https://docs.microsoft.com/azure/iot-edge/module-composition#define-or-update-desired-properties).

7. Seleccione **guardar** y repita el paso **5**.  

8. En el diálogo Módulo personalizado de IoT Edge use `opcpublisher` como nombre del módulo y, después, especifique el *URI de la imagen* del contenedor como 

   ```bash
   mcr.microsoft.com/iotedge/opc-publisher:latest
   ```

   Como *Opciones de creación del contenedor*, use el siguiente JSON:

   ```json
   {"Hostname":"publisher","Cmd":["publisher","--pf=./pn.json","--di=60","--to","--aa","--si=0","--ms=0"],"ExposedPorts":{"62222/tcp":{}},"HostConfig":{"PortBindings":{"62222/tcp":[{"HostPort":"62222"}] }}}
   ```

9. Seleccione **Guardar** y, después, **Siguiente** para ir a la sección de rutas.

10. En la pestaña rutas, pegue el siguiente 

    ```json
    {
      "routes": {
        "opctwinToIoTHub": "FROM /messages/modules/opctwin/* INTO $upstream",
        "opcpublisherToIoTHub": "FROM /messages/modules/opcpublisher/* INTO $upstream"
      }
    }
    ```

    y seleccione **Siguiente**

11. Revise la información de implementación y el manifiesto.  Debería tener el aspecto del manifiesto de implementación anterior.  Seleccione **Submit** (Enviar).

12. Una vez que los módulos se han implementado en el dispositivo, puede verlos todos en la página **Detalles del dispositivo** del portal. Esta página muestra el nombre de cada módulo implementado, así como información útil, como el estado de la implementación y el código de salida.

## <a name="deploying-using-azure-cli"></a>Implementación con la CLI de Azure

### <a name="prerequisites"></a>Prerequisites

1. Instale la versión más reciente de la [interfaz de la línea de comandos de Azure (AZ)](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) desde [aquí](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

### <a name="quickstart"></a>Guía de inicio rápido

1. Guarde el manifiesto de implementación anterior en un archivo `deployment.json`.  

2. Utilice el siguiente comando para aplicar la configuración a un dispositivo IoT Edge:

   ```azurecli
   az iot edge set-modules --device-id [device id] --hub-name [hub name] --content ./deployment.json
   ```

   El parámetro `device id` distingue mayúsculas y minúsculas. El parámetro "content" apunta al archivo del manifiesto de implementación que guardó. 
    ![az IoT Edge set-modules output](https://docs.microsoft.com/azure/iot-edge/media/how-to-deploy-cli/set-modules.png)

3. Una vez que los módulos se han implementado en el dispositivo, puede verlos todos con el siguiente comando:

   ```azurecli
   az iot hub module-identity list --device-id [device id] --hub-name [hub name]
   ```

   El parámetro del id. de dispositivo distingue mayúsculas y minúsculas. ![az iot hub module-identity list output](https://docs.microsoft.com/azure/iot-edge/media/how-to-deploy-cli/list-modules.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido cómo implementar OPC Twin desde cero, este es el siguiente paso que se le sugiere:

> [!div class="nextstepaction"]
> [Implementación de OPC Twin en un proyecto ya existente](howto-opc-twin-deploy-existing.md)
