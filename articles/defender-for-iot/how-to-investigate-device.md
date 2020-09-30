---
title: Investigación de un dispositivo sospechoso
description: En esta guía de procedimientos se explica cómo usar Defender for IoT para investigar un dispositivo IoT sospechoso mediante Log Analytics.
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
ms.date: 09/04/2020
ms.author: mlottner
ms.openlocfilehash: cd7802fa4a88c3b05140d30ab4d8b2bbd4adc8f4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932394"
---
# <a name="investigate-a-suspicious-iot-device"></a>Investigación de un dispositivo de IoT sospechoso

Las alertas del servicio Defender for IoT aportan claros indicios cuando existe la sospecha de que un dispositivo IoT ha participado en actividades sospechosas, o cuando hay señales que indican que un dispositivo está en peligro.

En esta guía, use las sugerencias de investigación que se proporcionan para ayudar a determinar los riesgos potenciales para su organización, a decidir cómo corregirlos y a descubrir las mejores formas de evitar ataques similares en el futuro.

> [!div class="checklist"]
> * Busque los datos del dispositivo
> * Investigue el uso de consultas kql

## <a name="how-can-i-access-my-data"></a>¿Cómo puedo acceder a mis datos?

De forma predeterminada, Defender for IoT almacena las recomendaciones y las alertas de seguridad en el área de trabajo de Log Analytics. También puede elegir almacenar los datos de seguridad sin formato.

Para buscar el área de trabajo de Log Analytics para el almacenamiento de datos:

1. Abra IoT Hub.
1. En **Seguridad**, seleccione **Configuración** y, luego, **Recopilación de datos**.
1. Cambie los detalles de la configuración del área de trabajo de Log Analytics.
1. Haga clic en **Save**(Guardar).

Después de la configuración, realice las siguientes operaciones para acceder a los datos almacenados en el área de trabajo de Log Analytics:

1. Seleccione una alerta de Defender for IoT y haga clic en ella.
1. Haga clic en **Further investigation** (Investigación adicional).
1. Seleccione **To see which devices have this alert click here and view the DeviceId column** (Para ver los dispositivos que tienen esta alerta haga clic aquí y vea la columna DeviceId).

## <a name="investigation-steps-for-suspicious-iot-devices"></a>Pasos de la investigación de dispositivos de IoT sospechosos

Si quiere consultar información y datos sin procesar relativos a sus dispositivos IoT, vaya a su área de trabajo de Log Analytics [para acceder a los datos](#how-can-i-access-my-data).

Vea las siguientes consultas kql de ejemplo para empezar a investigar alertas y actividades en el dispositivo.

### <a name="related-alerts"></a>Alertas relacionadas

Para averiguar si se desencadenaron otras alertas aproximadamente a la misma hora, utilice la siguiente consulta kql:

  ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityAlert
  | where ExtendedProperties contains device and ResourceId contains tolower(hub)
  | project TimeGenerated, AlertName, AlertSeverity, Description, ExtendedProperties
  ```

### <a name="users-with-access"></a>Usuarios con acceso

Para averiguar qué usuarios tienen acceso a este dispositivo, use la siguiente consulta kql:

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "LocalUsers"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     GroupNames=extractjson("$.GroupNames", EventDetails, typeof(string)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string))
  | summarize FirstObserved=min(TimestampLocal) by GroupNames, UserName
 ```
Use estos datos para detectar:

- Qué usuarios tienen acceso al dispositivo.
- ¿Los usuarios con acceso tienen los niveles de permiso esperados?

### <a name="open-ports"></a>Abrir puertos

Para averiguar qué puertos del dispositivo están actualmente en uso o se han usado, utilice la siguiente consulta kql:

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ListeningPorts"
     and extractjson("$.LocalPort", EventDetails, typeof(int)) <= 1024 // avoid short-lived TCP ports (Ephemeral)
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Protocol=extractjson("$.Protocol", EventDetails, typeof(string)),
     LocalAddress=extractjson("$.LocalAddress", EventDetails, typeof(string)),
     LocalPort=extractjson("$.LocalPort", EventDetails, typeof(int)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     RemotePort=extractjson("$.RemotePort", EventDetails, typeof(string))
  | summarize MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), AllowedRemoteIPAddress=makeset(RemoteAddress), AllowedRemotePort=makeset(RemotePort) by Protocol, LocalPort
 ```

Use estos datos para detectar:

- Qué sockets de escucha están actualmente activos en el dispositivo.
- Si los sockets de escucha activos actualmente deberían estar permitidos.
- Si hay alguna dirección remota sospechosa conectada al dispositivo.

### <a name="user-logins"></a>Inicios de sesión de usuario

Para averiguar qué usuarios han iniciado sesión en el dispositivo, use la siguiente consulta kql:

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "Login"
     // filter out local, invalid and failed logins
     and EventDetails contains "RemoteAddress"
     and EventDetails !contains '"RemoteAddress":"127.0.0.1"'
     and EventDetails !contains '"UserName":"(invalid user)"'
     and EventDetails !contains '"UserName":"(unknown user)"'
     //and EventDetails !contains '"Result":"Fail"'
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string)),
     LoginHandler=extractjson("$.Executable", EventDetails, typeof(string)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     Result=extractjson("$.Result", EventDetails, typeof(string))
  | summarize CntLoginAttempts=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), CntIPAddress=dcount(RemoteAddress), IPAddress=makeset(RemoteAddress) by UserName, Result, LoginHandler
 ```

Utilice los resultados de la consulta para detectar:

- Qué usuarios han iniciado sesión el dispositivo.
- Si los usuarios que ha iniciado sesión deben iniciar sesión teóricamente.
- Si los usuarios que han iniciado sesión se han conectado desde direcciones IP esperadas o inesperadas.

### <a name="process-list"></a>Lista de procesos

Para averiguar si la lista de procesos es la prevista, use la siguiente consulta kql:

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ProcessCreate"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Executable=extractjson("$.Executable", EventDetails, typeof(string)),
     UserId=extractjson("$.UserId", EventDetails, typeof(string)),
     CommandLine=extractjson("$.CommandLine", EventDetails, typeof(string))
  | join kind=leftouter (
     // user UserId details
     SecurityIoTRawEvent
     | where
        DeviceId == device and AssociatedResourceId contains tolower(hub)
        and RawEventName == "LocalUsers"
     | project
        UserId=extractjson("$.UserId", EventDetails, typeof(string)),
        UserName=extractjson("$.UserName", EventDetails, typeof(string))
     | distinct UserId, UserName
  ) on UserId
  | extend UserIdName = strcat("Id:", UserId, ", Name:", UserName)
  | summarize CntExecutions=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), ExecutingUsers=makeset(UserIdName), ExecutionCommandLines=makeset(CommandLine) by Executable
```

Utilice los resultados de la consulta para detectar:

- Si se ejecutó algún proceso en el dispositivo.
- Si los procesos los han ejecutado los usuarios adecuados.
- Si las ejecuciones de la línea de comandos contenían los argumentos correctos y esperados.

## <a name="next-steps"></a>Pasos siguientes

Tras investigar un dispositivo y conocer mejor los riesgos, es posible que desee considerar la posibilidad de [configurar alertas personalizadas](quickstart-create-custom-alerts.md) para mejorar la postura de seguridad de su solución de IoT. Si aún no tiene un agente de dispositivos, considere la posibilidad de [implementar un agente de seguridad](how-to-deploy-agent.md) o de [cambiar la configuración de un agente de dispositivos existente](how-to-agent-configuration.md) para mejorar los resultados.
