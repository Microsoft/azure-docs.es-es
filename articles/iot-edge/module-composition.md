---
title: 'Implementación de módulos y rutas con manifiestos de implementación: Azure IoT Edge'
description: Obtenga información sobre cómo un manifiesto de implementación declara qué módulos se deben implementar, cómo implementarlos y cómo crear rutas de mensajes entre ellos.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/26/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: dd2b9bc462a9d4bc11f49a7e3294e52f88a926fb
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86511847"
---
# <a name="learn-how-to-deploy-modules-and-establish-routes-in-iot-edge"></a>Obtenga información sobre cómo implementar módulos y establecer rutas en IoT Edge

Cada dispositivo IoT Edge ejecuta al menos dos módulos: $edgeAgent y $edgeHub, que constituyen el entorno de ejecución de Azure IoT Edge. El dispositivo IoT Edge puede ejecutar varios módulos adicionales para llevar a cabo cualquier número de procesos. Use un manifiesto de implementación para indicar al dispositivo qué módulos debe instalar y cómo configurarlos para que funcionen conjuntamente.

El *manifiesto de implementación* es un documento JSON que describe lo siguiente:

* El módulo gemelo del **agente de IoT Edge**, que incluye tres componentes:
  * La imagen de contenedor para cada módulo que se ejecuta en el dispositivo.
  * Las credenciales para tener acceso a registros privados del contenedor que contienen imágenes del módulo.
  * Instrucciones sobre cómo se debe crearse y administrar cada módulo.
* El módulo gemelo del **centro de IoT Edge**, que incluye cómo fluyen los mensajes entre los módulos y finalmente hacia IoT Hub.
* Las propiedades deseadas de cualquier módulo gemelo adicional (opcional).

Todos los dispositivos IoT Edge deben configurarse con un manifiesto de implementación. Si no, una instancia de IoT Edge en tiempo de ejecución recién instalada notificará un código de error hasta que se configuren con un manifiesto válido.

En los tutoriales de Azure IoT Edge, creará un manifiesto de implementación a través de un asistente del portal de Azure IoT Edge. También puede aplicar un manifiesto de implementación mediante programación con REST o el SDK del servicio IoT Hub. Para más información, consulte el artículo [Descripción de las implementaciones de IoT Edge](module-deployment-monitoring.md).

## <a name="create-a-deployment-manifest"></a>Creación de un manifiesto de implementación

A nivel general, un manifiesto de implementación es una lista de módulos gemelos que se configuran con sus propiedades deseadas. Un manifiesto de implementación indica a un dispositivo IoT Edge (o a un grupo de dispositivos) qué módulos debe instalar y cómo configurarlos. Los manifiestos de implementación incluyen las *propiedades deseadas* de cada módulo gemelo. Los dispositivos IoT Edge informan sobre las *propiedades notificadas* de cada módulo.

Se requieren dos módulos en cada manifiesto de implementación: `$edgeAgent` y `$edgeHub`. Estos módulos forman parte del entorno de ejecución de Azure IoT Edge que administra el dispositivo IoT Edge y los módulos que se ejecutan en él. Para más información acerca de estos módulos, consulte [Información sobre el entorno de ejecución de Azure IoT Edge y su arquitectura](iot-edge-runtime.md).

Además de los dos módulos en tiempo de ejecución, puede agregar hasta 30 módulos de su elección para que se ejecuten en un dispositivo IoT Edge.

Un manifiesto de implementación que contiene solo el entorno de ejecución de Azure IoT Edge (edgeAgent y edgeHub) es válido.

Los manifiestos de implementación siguen esta estructura:

```json
{
    "modulesContent": {
        "$edgeAgent": { // required
            "properties.desired": {
                // desired properties of the Edge agent
                // includes the image URIs of all modules
                // includes container registry credentials
            }
        },
        "$edgeHub": { //required
            "properties.desired": {
                // desired properties of the Edge hub
                // includes the routing information between modules, and to IoT Hub
            }
        },
        "module1": {  // optional
            "properties.desired": {
                // desired properties of module1
            }
        },
        "module2": {  // optional
            "properties.desired": {
                // desired properties of module2
            }
        },
        ...
    }
}
```

## <a name="configure-modules"></a>Configuración de módulos

Defina cómo instala el entorno de ejecución de Azure IoT Edge los módulos en la implementación. El agente de IoT Edge es el componente del entorno de ejecución que administra la instalación, las actualizaciones y los informes de estado de un dispositivo IoT Edge. Por tanto, el módulo gemelo $edgeAgent contiene la información de configuración y administración de todos los módulos. Esta información incluye los parámetros de configuración para el propio agente de IoT Edge.

Para ver una lista completa de propiedades que pueden o deben incluirse, consulte [Properties of the IoT Edge agent and IoT Edge hub](module-edgeagent-edgehub.md) (Propiedades del agente de IoT Edge y del centro de IoT Edge).

Las propiedades de $edgeAgent siguen esta estructura:

```json
"$edgeAgent": {
    "properties.desired": {
        "schemaVersion": "1.0",
        "runtime": {
            "settings":{
                "registryCredentials":{ // give the edge agent access to container images that aren't public
                    }
                }
            }
        },
        "systemModules": {
            "edgeAgent": {
                // configuration and management details
            },
            "edgeHub": {
                // configuration and management details
            }
        },
        "modules": {
            "module1": { // optional
                // configuration and management details
            },
            "module2": { // optional
                // configuration and management details
            }
        }
    }
},
```

## <a name="declare-routes"></a>Declaración de rutas

El centro de IoT Edge administra la comunicación entre los módulos, IoT Hub y todos los dispositivos de hoja. Por tanto, el módulo gemelo $edgeHub contiene una propiedad deseada denominada *routes* que declara cómo se pasan los mensajes en una implementación. Puede tener varias rutas dentro de la misma implementación.

Las rutas se declaran en las propiedades deseadas de **$edgeHub** con la sintaxis siguiente:

```json
"$edgeHub": {
    "properties.desired": {
        "routes": {
            "route1": "FROM <source> WHERE <condition> INTO <sink>",
            "route2": "FROM <source> WHERE <condition> INTO <sink>"
        },
    }
}
```

Cada ruta necesita un origen y un receptor, pero la condición es una parte opcional que puede usar para filtrar los mensajes.

### <a name="source"></a>Source

El origen especifica de dónde proceden los mensajes. IoT Edge puede enrutar los mensajes desde los dispositivos de hoja o desde los módulos.

Mediante el SDK de IoT, los módulos pueden declarar colas de salida específicas para sus mensajes mediante la clase ModuleClient. Las colas de salida no son necesarias, pero son útiles para administrar varias rutas. Los dispositivos de hoja pueden usar la clase DeviceClient de los SDK de IoT para enviar mensajes a dispositivos de puerta de enlace de IoT Edge de la misma manera que enviarían mensajes a IoT Hub. Para más información, consulte [Información y uso de los SDK de Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md).

La propiedad de origen puede ser cualquiera de los siguientes valores:

| Source | Descripción |
| ------ | ----------- |
| `/*` | Todos los mensajes del dispositivo a la nube o las notificaciones de cambios de gemelos de cualquier módulo o dispositivo de hoja. |
| `/twinChangeNotifications` | Todos los cambios de gemelos (propiedades notificadas) procedentes de todos los módulos o dispositivos de hoja. |
| `/messages/*` | Cualquier mensaje de dispositivo a nube que envíe un dispositivo de hoja o un módulo con alguna salida o sin ninguna. |
| `/messages/modules/*` | Cualquier mensaje de dispositivo a nube que envíe un módulo con alguna salida o sin ninguna. |
| `/messages/modules/<moduleId>/*` | Cualquier mensaje de dispositivo a nube que envíe un módulo específico con alguna salida o sin ninguna. |
| `/messages/modules/<moduleId>/outputs/*` | Cualquier mensaje de dispositivo a nube que envíe un módulo específico mediante alguna salida. |
| `/messages/modules/<moduleId>/outputs/<output>` | Cualquier mensaje de dispositivo a nube que envíe un módulo específico mediante una salida específica. |

### <a name="condition"></a>Condición

La condición es opcional en una declaración de ruta. Si desea pasar todos los mensajes desde el receptor al origen, omita la cláusula **WHERE** por completo. O bien, puede usar el [lenguaje de consulta de IoT Hub](../iot-hub/iot-hub-devguide-routing-query-syntax.md) para filtrar por determinados mensajes o tipos de mensajes que cumplen la condición. Las rutas de IoT Edge no admiten mensajes de filtrado basados en etiquetas o propiedades de gemelos.

Los mensajes que pasan entre módulos con IoT Edge tienen el mismo formato que los mensajes que pasan entre los dispositivos y Azure IoT Hub. Todos los mensajes tienen el formato JSON y tienen los parámetros **systemProperties**, **appProperties** y **body**.

Puede generar consultas en función de cualquiera de los tres parámetros con la sintaxis siguiente:

* Propiedades del sistema: `$<propertyName>` o `{$<propertyName>}`
* Propiedades de la aplicación: `<propertyName>`
* Propiedades del cuerpo: `$body.<propertyName>`

Para obtener ejemplos sobre cómo crear consultas para las propiedades de mensaje, consulte [Expresiones de consulta de rutas de mensajes de dispositivo a la nube](../iot-hub/iot-hub-devguide-routing-query-syntax.md).

Un ejemplo que es específico de IoT Edge es cuando desea filtrar los mensajes recibidos en un dispositivo de puerta de enlace desde un dispositivo hoja. Los mensajes que proceden de los módulos incluyen una propiedad del sistema denominada **connectionModuleId**. Así pues, si desea enrutar los mensajes desde dispositivos hoja directamente a IoT Hub, utilice la siguiente ruta para excluir mensajes de módulo:

```query
FROM /messages/* WHERE NOT IS_DEFINED($connectionModuleId) INTO $upstream
```

### <a name="sink"></a>Receptor

El receptor define dónde se envían los mensajes. Solo los módulos e IoT Hub pueden recibir mensajes. No se pueden enrutar mensajes a otros dispositivos. No hay opción de carácter comodín en la propiedad del receptor.

La propiedad del receptor puede ser cualquiera de los siguientes valores:

| Receptor | Descripción |
| ---- | ----------- |
| `$upstream` | Envía el mensaje a IoT Hub. |
| `BrokeredEndpoint("/modules/<moduleId>/inputs/<input>")` | Envía el mensaje a una entrada específica de un módulo específico. |

IoT Edge proporciona garantías de por lo menos una vez. El centro de IoT Edge almacena mensajes localmente en caso de que una ruta no pueda entregar el mensaje a su receptor. Por ejemplo, si el centro de IoT Edge no se puede conectar a IoT Hub o el módulo de destino no está conectado.

El centro de IoT Edge almacena los mensajes durante el tiempo especificado en la propiedad `storeAndForwardConfiguration.timeToLiveSecs` de las [propiedades deseadas del centro de IoT Edge](module-edgeagent-edgehub.md).

## <a name="define-or-update-desired-properties"></a>Definición o actualización de las propiedades deseadas

El manifiesto de implementación especifica las propiedades deseadas de cada módulo implementado en el dispositivo IoT Edge. Las propiedades deseadas del manifiesto de implementación sobrescriben a las que estén presentes en el módulo gemelo.

Si no se especifican las propiedades deseadas del módulo gemelo en el manifiesto de implementación, IoT Hub no modificará el módulo. En su lugar, se podrán establecer las propiedades deseadas mediante programación.

Los módulos gemelos se modifican con los mismos mecanismos que los dispositivos gemelos. Para más información vea la [guía para desarrolladores de módulos gemelos](../iot-hub/iot-hub-devguide-module-twins.md).

## <a name="deployment-manifest-example"></a>Ejemplo de manifiesto de implementación

El ejemplo siguiente muestra el aspecto de un documento de manifiesto de implementación válido.

```json
{
  "modulesContent": {
    "$edgeAgent": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "runtime": {
          "type": "docker",
          "settings": {
            "minDockerVersion": "v1.25",
            "loggingOptions": "",
            "registryCredentials": {
              "ContosoRegistry": {
                "username": "myacr",
                "password": "<password>",
                "address": "myacr.azurecr.io"
              }
            }
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
              "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
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
          },
          "filtermodule": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "env": {
              "tempLimit": {"value": "100"}
            },
            "settings": {
              "image": "myacr.azurecr.io/filtermodule:latest",
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
          "sensorToFilter": "FROM /messages/modules/SimulatedTemperatureSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/filtermodule/inputs/input1\")",
          "filterToIoTHub": "FROM /messages/modules/filtermodule/outputs/output1 INTO $upstream"
        },
        "storeAndForwardConfiguration": {
          "timeToLiveSecs": 10
        }
      }
    }
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una lista completa de propiedades que pueden o deben incluirse en $edgeAgent y $edgeHub, consulte [Properties of the IoT Edge agent and IoT Edge hub](module-edgeagent-edgehub.md) (Propiedades del agente de IoT Edge y el centro de IoT Edge).

* Ahora que sabe cómo se usan los módulos de IoT Hub, [descubra los requisitos y las herramientas para desarrollar módulos de IoT Edge](module-development.md).
