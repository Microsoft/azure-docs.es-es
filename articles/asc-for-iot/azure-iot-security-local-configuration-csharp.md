---
title: Configuraciones local del agente de seguridad (C#)
description: Obtenga más información sobre el archivo de configuración local del agente de seguridad y del servicio de seguridad Azure Security Center para IoT en C#.
services: asc-for-iot
ms.service: asc-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 2cf6a49b-5d35-491f-abc3-63ec24eb4bc2
ms.subservice: asc-for-iot
ms.devlang: na
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/26/2019
ms.author: mlottner
ms.openlocfilehash: eab7764ede300ca355ec6193312279c5726f8a01
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212410"
---
# <a name="understanding-the-local-configuration-file-c-agent"></a>Información sobre el archivo de configuración local (agente de C#)

El agente de seguridad de Azure Security Center para IoT usa configuraciones de un archivo de configuración local.

El agente de seguridad lee el archivo de configuración una vez cuando se inicia el agente. Las configuraciones que se encuentran en el archivo de configuración local contienen la configuración de autenticación y otras configuraciones relacionadas con el agente.

El agente de seguridad de C# utiliza varios archivos de configuración:

- **General.config**: configuraciones relacionadas con el agente.
- **Authentication.config**: configuración relacionada con la autenticación (incluidos los detalles de autenticación).
- **SecurityIotInterface.config**: configuraciones relacionadas con IoT.

Los archivos de configuración contienen la configuración predeterminada. La configuración de autenticación se rellena durante la instalación del agente, y se realizan cambios en el archivo de configuración cuando se reinicia el agente.

## <a name="configuration-file-location"></a>Ubicación del archivo de configuración

Para Linux:

- Los archivos de configuración del sistema operativo se encuentran en `/var/ASCIoTAgent`.

Para Windows:

- Los archivos de configuración del sistema operativo se encuentran en el directorio del agente de seguridad.

### <a name="generalconfig-configurations"></a>Configuraciones de General.config

| Nombre de la configuración | Valores posibles | Detalles |
|:-----------|:---------------|:--------|
| agentId | GUID | Identificador único del agente |
| readRemoteConfigurationTimeout | TimeSpan | Período de tiempo para capturar la configuración remota de IoT Hub. Si el agente no puede recuperar la configuración en el tiempo especificado, se agotará el tiempo de espera de la operación.|
| schedulerInterval | TimeSpan | Intervalo del programador interno. |
| producerInterval | TimeSpan | Intervalo de trabajo del productor de eventos. |
| consumerInterval | TimeSpan | Intervalo de trabajo del consumidor de eventos. |
| highPriorityQueueSizePercentage | 0 < número < 1 | La parte de la memoria caché total dedicada a los mensajes de prioridad alta. |
| logLevel | "Off", "Fatal", "Error", "Warning", "Information" y "Debug"  | Los mensajes de registro iguales y superiores a esta gravedad se registran en la consola de depuración (Syslog en Linux). |
| fileLogLevel |  "Off", "Fatal", "Error", "Warning", "Information" y "Debug"| Los mensajes de registro iguales y superiores a esta gravedad se registran en el archivo (Syslog en Linux). |
| diagnosticVerbosityLevel | "None", "Some" y "All", | Nivel de detalle de los eventos de diagnóstico. None: no se envían eventos de diagnóstico, Some: solo se envían los eventos de diagnóstico con importancia alta y All: todos los registros también se envían como eventos de diagnóstico. |
| logFilePath | Ruta a archivo | Si fileLogLevel > Off, los registros se escriben en este archivo. |
| defaultEventPriority | "High", "Low" y "Off" | Prioridad del evento predeterminada. |

### <a name="generalconfig-example"></a>Ejemplo de General.config

```xml
<?xml version="1.0" encoding="utf-8"?>
<General>
  <add key="agentId" value="da00006c-dae9-4273-9abc-bcb7b7b4a987" />
  <add key="readRemoteConfigurationTimeout" value="00:00:30" />
  <add key="schedulerInterval" value="00:00:01" />
  <add key="producerInterval" value="00:02:00" />
  <add key="consumerInterval" value="00:02:00" />
  <add key="highPriorityQueueSizePercentage" value="0.5" />
  <add key="logLevel" value="Information" />
  <add key="fileLogLevel" value="Off"/>
  <add key="diagnosticVerbosityLevel" value="Some" />
  <add key="logFilePath" value="IotAgentLog.log" />
  <add key="defaultEventPriority" value="Low"/>
</General>
```

### <a name="authenticationconfig"></a>Authentication.config

| Nombre de la configuración | Valores posibles | Detalles |
|:-----------|:---------------|:--------|
| moduleName | string | Nombre de la identidad del módulo de seguridad. Este nombre debe corresponderse con el nombre de identidad del módulo del dispositivo. |
| deviceId | string | Identificador del dispositivo (como está registrado en Azure IoT Hub). || schedulerInterval | Cadena TimeSpan | Intervalo del programador interno. |
| gatewayHostname | string | Nombre de host de Azure IoT Hub. Normalmente, <my-hub>.azure-devices.net |
| filePath | string - path to file | Ruta de acceso al archivo que contiene el secreto de autenticación.|
| type | "SymmetricKey" y "SelfSignedCertificate" | Secreto de usuario para la autenticación. Elija *SymmetricKey* si el secreto de usuario es una clave simétrica y seleccione *self-signed certificate* si el secreto es un certificado autofirmado. |
| identity | "DPS", "Module" y "Device" | Identidad de autenticación: DPS si la autenticación se realiza a través de DPS, Module si la autenticación se realiza mediante las credenciales del módulo o Device si la autenticación se realiza con las credenciales del dispositivo.
| certificateLocationKind |  "LocalFile" y "Store" | LocalFile si el certificado se almacena en un archivo y Store si el certificado se encuentra en un almacén de certificados. |
| idScope | string | Ámbito del identificador de DPS |
| registrationId | string  | Id. de registro del dispositivo de DPS. |
|

### <a name="authenticationconfig-example"></a>Ejemplo de Authentication.config

```xml
<?xml version="1.0" encoding="utf-8"?>
<Authentication>
  <add key="moduleName" value="azureiotsecurity"/>
  <add key="deviceId" value="d1"/>
  <add key="gatewayHostname" value=""/>
  <add key="filePath" value="c:\p-dps-d1.pfx"/>
  <add key="type" value="SelfSignedCertificate" />                     <!-- SymmetricKey, SelfSignedCertificate-->
  <add key="identity" value="DPS" />                 <!-- Device, Module, DPS -->
  <add key="certificateLocationKind" value="LocalFile" />  <!-- LocalFile, Store -->
  <add key="idScope" value="0ne0005335B"/>
  <add key="registrationId" value="d1"/>
</Authentication>
```

### <a name="securityiotinterfaceconfig"></a>SecurityIotInterface.config

| Nombre de la configuración | Valores posibles | Detalles |
|:-----------|:---------------|:--------|
| transportType | "Ampq" y "Mqtt" | Tipo de transporte de IoT Hub. |
|

### <a name="securityiotinterfaceconfig-example"></a>Ejemplo de SecurityIotInterface.config

```xml
<ExternalInterface>
  <add key="facadeType"  value="Microsoft.Azure.Security.IoT.Agent.Common.SecurityIoTHubInterface, Security.Common" />
  <add key="transportType" value="Amqp"/>
</ExternalInterface>
```

## <a name="next-steps"></a>Pasos siguientes

- Lea la [introducción](overview.md) del servicio Azure Security Center para IoT
- Obtenga más información sobre la [arquitectura](architecture.md) de Azure Security Center para IoT
- Habilite el [servicio](quickstart-onboard-iot-hub.md) Azure Security Center para IoT
- Lea las [preguntas frecuentes](resources-frequently-asked-questions.md) del servicio Azure Security Center para IoT
- Aprenda a acceder a [datos de seguridad sin procesar](how-to-security-data-access.md)
- Obtenga información acerca de las [recomendaciones](concept-recommendations.md)
- Obtenga información sobre las [alertas de seguridad](concept-security-alerts.md)
