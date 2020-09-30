---
title: Configuraciones local del agente de seguridad (C)
description: Obtenga información sobre Defender para las configuraciones locales del agente de C.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/26/2019
ms.author: mlottner
ms.openlocfilehash: 7cc6886b5341d8bc8a82288ad8a2a699381a953c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931979"
---
# <a name="understanding-the-localconfigurationjson-file---c-agent"></a>Información acerca del archivo LocalConfiguration.json: agente de C

El agente de seguridad de Defender para IoT usa configuraciones de un archivo de configuración local.
El agente de seguridad lee el archivo de configuración una vez cuando se inicia el agente.
La configuración que se encuentran en el archivo de configuración local contienen la configuración de autenticación y otras relacionadas con el agente.
El archivo contiene configuraciones en pares de "clave-valor" en notación JSON, y cuando se instala el agente las configuraciones se rellenan.

De forma predeterminada, el archivo se encuentra en:/var/ASCIoTAgent/LocalConfiguration.json

Los cambios en el archivo de configuración tienen lugar cuando se reinicia el agente.

## <a name="security-agent-configurations-for-c"></a>Configuraciones del agente de seguridad para C

| Nombre de la configuración | Valores posibles | Detalles |
|:-----------|:---------------|:--------|
| AgentId | GUID | El identificador único del agente |
| TriggerdEventsInterval | Cadena ISO8601 | Intervalo del programador para la recopilación de eventos desencadenados |
| ConnectionTimeout | Cadena ISO8601 | Período de tiempo antes de que se agote el tiempo de espera de la conexión a IoThub |
| Authentication | JsonObject | Configuración de autenticación. Este objeto contiene toda la información necesaria para la autenticación en IoTHub |
| Identidad | "DPS", "SecurityModule", "Device" | Identidad de autenticación: DPS si la autenticación se realiza con DPS, SecurityModule si la autenticación se realiza mediante las credenciales del módulo de seguridad, o Device si la autenticación se realiza con las credenciales del dispositivo |
| AuthenticationMethod | "SasToken", "SelfSignedCertificate" | El secreto de usuario para la autenticación: elija SasToken si el secreto utilizado es una clave simétrica, seleccione SelfSignedCertificate si el secreto es un certificado autofirmado  |
| FilePath | Ruta de acceso al archivo (cadena) | Ruta de acceso al archivo que contiene el secreto de autenticación |
| HostName | string | Nombre del host de la instancia de Azure IoT Hub. Normalmente, <my-hub>.azure-devices.net |
| deviceId | string | El identificador del dispositivo (como está registrado en Azure IoT Hub) |
| DPS | JsonObject | Configuraciones relacionadas con DPS |
| IDScope | string | Ámbito del identificador de DPS |
| RegistrationId | string  | Identificador del registro de dispositivo de DPS |
| Registro | JsonObject | Configuraciones relacionadas con el registrador de agente |
| SystemLoggerMinimumSeverity | 0 <= número <= 4 | Los mensajes de registro iguales y superiores a esta gravedad se registrarán en/var/log/syslog (0 es la gravedad más baja) |
| DiagnosticEventMinimumSeverity | 0 <= número <= 4 | Los mensajes de registro iguales y superiores a esta gravedad se enviarán como eventos de diagnóstico (0 es la gravedad más baja) |

## <a name="security-agent-configurations-code-example"></a>Ejemplo de código de configuraciones del agente de seguridad

```json
{
    "Configuration" : {
        "AgentId" : "b97faf0a-0f57-471f-9dab-46a8e1764946",
        "TriggerdEventsInterval" : "PT2M",
        "ConnectionTimeout" : "PT30S",
        "Authentication" : {
            "Identity" : "Device",
            "AuthenticationMethod" : "SasToken",
            "FilePath" : "/path/to/my/SymmetricKey",
            "DeviceId" : "my-device",
            "HostName" : "my-iothub.azure-devices.net",
            "DPS" : {
                "IDScope" : "",
                "RegistrationId" : ""
            }
        },
        "Logging": {
            "SystemLoggerMinimumSeverity": 0,
            "DiagnoticEventMinimumSeverity": 2
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

- Lea la [información general](overview.md) del servicio Defender para IoT.
- Más información sobre la [arquitectura](architecture.md) de Defender para IoT.
- Habilite el [servicio](quickstart-onboard-iot-hub.md) Defender para IoT.
- Lea las [preguntas más frecuentes](resources-frequently-asked-questions.md) del servicio Defender para IoT.
- Aprenda a acceder a [datos de seguridad sin procesar](how-to-security-data-access.md)
- Obtenga información acerca de las [recomendaciones](concept-recommendations.md)
- Obtenga información sobre las [alertas de seguridad](concept-security-alerts.md)
