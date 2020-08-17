---
title: Creación de webhooks para reglas en Azure IoT Central | Microsoft Docs
description: Cree webhooks en Azure IoT Central para notificar automáticamente a otras aplicaciones cuándo se activan las reglas.
author: viv-liu
ms.author: viviali
ms.date: 04/03/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: corywink
ms.openlocfilehash: b2ac4bbf1457144d23a91c4e83b554b3ee806119
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337235"
---
# <a name="create-webhook-actions-on-rules-in-azure-iot-central"></a>Creación de acciones de webhook para reglas en Azure IoT Central

*Este tema se aplica a los compiladores y administradores.*

Los webhooks permiten conectar la aplicación de IoT Central a otras aplicaciones y servicios para supervisión remota y notificaciones. Los webhooks notifican automáticamente a otras aplicaciones y servicios a los que se conecta si se activa una regla en la aplicación IoT Central. Su aplicación IoT Central envía una solicitud POST al otro punto de conexión HTTP de la aplicación siempre que se desencadena una regla. La carga útil contiene detalles del dispositivo y del desencadenador de la regla.

## <a name="set-up-the-webhook"></a>Configuración del webhook

En este ejemplo, se conectará a RequestBin para obtener notificaciones mediante webhooks cuando se activan las reglas.

1. Abra [RequestBin](https://requestbin.net/).

1. Cree una instancia de RequestBin y copie la **dirección URL de la papelera**.

1. Cree una [regla de telemetría](tutorial-create-telemetry-rules.md). Guarde la regla y agregue una nueva acción.

    ![Pantalla de creación de webhooks](media/howto-create-webhooks/webhookcreate.png)

1. Elija la acción de webhook y proporcione un nombre para mostrar y pegue la dirección URL de la papelera como la dirección URL de devolución de llamada.

1. Guarde la regla.

Ahora, cuando se desencadena la regla, verá que aparece una nueva solicitud en RequestBin.

## <a name="payload"></a>Carga

Cuando se desencadena una regla, se realiza una solicitud HTTP POST a la dirección URL de devolución de llamada que contiene una carga útil JSON con los detalles de telemetría, dispositivo, regla y aplicación. La carga útil podría ser similar a la siguiente:

```json
{
    "timestamp": "2020-04-06T00:20:15.06Z",
    "action": {
        "id": "<id>",
        "type": "WebhookAction",
        "rules": [
            "<rule_id>"
        ],
        "displayName": "Webhook 1",
        "url": "<callback_url>"
    },
    "application": {
        "id": "<application_id>",
        "displayName": "Contoso",
        "subdomain": "contoso",
        "host": "contoso.azureiotcentral.com"
    },
    "device": {
        "id": "<device_id>",
        "etag": "<etag>",
        "displayName": "MXChip IoT DevKit - 1yl6vvhax6c",
        "instanceOf": "<device_template_id>",
        "simulated": true,
        "provisioned": true,
        "approved": true,
        "cloudProperties": {
            "City": {
                "value": "Seattle"
            }
        },
        "properties": {
            "deviceinfo": {
                "firmwareVersion": {
                    "value": "1.0.0"
                }
            }
        },
        "telemetry": {
            "<interface_instance_name>": {
                "humidity": {
                    "value": 47.33228889360127
                }
            }
        }
    },
    "rule": {
        "id": "<rule_id>",
        "displayName": "Humidity monitor"
    }
}
```
Si la regla supervisa la telemetría agregada durante un período de tiempo, la carga útil contendrá una sección de telemetría diferente.

```json
{
    "telemetry": {
        "<interface_instance_name>": {
            "Humidity": {
                "avg": 39.5
            }
        }
    }
}
```

## <a name="data-format-change-notice"></a>Aviso de cambio de formato de datos

Si tiene uno o varios webhooks creados y guardados antes del **3 de abril de 2020**, deberá eliminar el webhook y crear uno nuevo. Esto se debe a que los webhooks más antiguos usan un formato de carga anterior que se retirará en el futuro.

### <a name="webhook-payload-format-deprecated-as-of-3-april-2020"></a>Carga de webhook (formato en desuso a partir del 3 de abril de 2020)

```json
{
    "id": "<id>",
    "displayName": "Webhook 1",
    "timestamp": "2019-10-24T18:27:13.538Z",
    "rule": {
        "id": "<id>",
        "displayName": "High temp alert",
        "enabled": true
    },
    "device": {
        "id": "mx1",
        "displayName": "MXChip IoT DevKit - mx1",
        "instanceOf": "<device-template-id>",
        "simulated": true,
        "provisioned": true,
        "approved": true
    },
    "data": [{
        "@id": "<id>",
        "@type": ["Telemetry"],
        "name": "temperature",
        "displayName": "Temperature",
        "value": 66.27310467496761,
        "interfaceInstanceName": "sensors"
    }],
    "application": {
        "id": "<id>",
        "displayName": "x - Store Analytics Checkout",
        "subdomain": "<subdomain>",
        "host": "<host>"
    }
}
```

## <a name="known-limitations"></a>Restricciones conocidas

Actualmente no hay ningún mecanismo de programación para suscribir o cancelar la suscripción de estos webhooks mediante una API.

Si tiene ideas sobre cómo mejorar esta característica, envíe sus sugerencias a nuestro [foro de UserVoice](https://feedback.azure.com/forums/911455-azure-iot-central).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a configurar y usar webhooks, el siguiente paso sugerido es explorar la [configuración de grupos de acciones de Azure Monitor](howto-use-action-groups.md).
