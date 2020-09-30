---
title: Métodos de autenticación del agente de seguridad
description: Obtenga información sobre los distintos métodos de autenticación disponibles al usar el servicio Defender para IoT.
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
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 896baa70fb6e18165a025459a063e33d705261ba
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932302"
---
# <a name="security-agent-authentication-methods"></a>Métodos de autenticación del agente de seguridad

En este artículo se explican los diferentes métodos de autenticación que se pueden utilizar con el agente AzureIoTSecurity para autenticarse en IoT Hub.

Se requiere un módulo de seguridad por cada dispositivo incorporado a Defender para IoT en IoT Hub. Para autenticar el dispositivo, Defender para IoT puede usar uno de dos métodos posibles. Elija el método que más le convenga en su solución de IoT existente.

> [!div class="checklist"]
> * Opción SecurityModule
> * Opción de dispositivo

## <a name="authentication-methods"></a>Métodos de autenticación

Los dos métodos para que el agente AzureIoTSecurity realice la autenticación son:

- Modo de autenticación **SecurityModule**<br>
El agente se autentica con la identidad del módulo de seguridad independientemente de la identidad del dispositivo.
Use este tipo de autenticación si quiere que el agente de seguridad use un método de autenticación dedicado a través del módulo de seguridad (solo clave simétrica).

- Modo de autenticación de **dispositivo**<br>
En este método, el agente de seguridad se autentica primero con la identidad del dispositivo. Después de la autenticación inicial, el agente de Defender para IoT hace una llamada **REST** a IoT Hub mediante la API REST con los datos de autenticación del dispositivo. Después, el agente de Defender para IoT solicita los datos y el método de autenticación del módulo de seguridad a IoT Hub. En el último paso, el agente de Defender para IoT realiza una autenticación en el módulo de Defender para IoT.

Emplee este tipo de autenticación si quiere que el agente de seguridad reutilice un método de autenticación de dispositivo dedicado ya existente (certificado autofirmado o clave simétrica).

Vea [Parámetros de instalación del agente de seguridad](#security-agent-installation-parameters) para saber cómo configurarlo.

## <a name="authentication-methods-known-limitations"></a>Limitaciones conocidas de los métodos de autenticación

- El modo de autenticación **SecurityModule** admite únicamente la autenticación mediante una clave simétrica.
- El modo de autenticación de **dispositivo** no admite certificados firmados por una entidad de certificación.

## <a name="security-agent-installation-parameters"></a>Parámetros de instalación del agente de seguridad

Al [implementar un agente de seguridad](how-to-deploy-agent.md), los detalles de autenticación se deben proporcionar como argumentos.
Estos argumentos se documentan en la siguiente tabla.

|Nombre de parámetro de Linux | Nombre de parámetro de Windows | Parámetro abreviado |Descripción|Opciones|
|---------------------|---------------|---------|---------------|---------------|
|authentication-identity|AuthenticationIdentity|aui|Identidad de autenticación| **SecurityModule** o **Device**|
|authentication-method|AuthenticationMethod|aum|Método de autenticación|**SymmetricKey** o **SelfSignedCertificate**|
|file-path|FilePath|f|Ruta de acceso absoluta completa del archivo que contiene el certificado o la clave simétrica| |
|host-name|HostName|hn|FQDN de IoT Hub|Ejemplo: ContosoIotHub.azure-devices.net|
|device-id|deviceId|di|Id. de dispositivo|Ejemplo: MyDevice1|
|certificate-location-kind|CertificateLocationKind|cl|Ubicación del almacén de certificados|**LocalFile** o **Store**|
|

Cuando se usa el script de instalación del agente de seguridad, la siguiente configuración se realiza automáticamente. Para modificar manualmente la autenticación del agente de seguridad, edite el archivo de configuración.

## <a name="change-authentication-method-after-deployment"></a>Cambiar el método de autenticación después de la implementación

Al implementar a un agente de seguridad con un script de instalación, se crea automáticamente un archivo de configuración.

Para cambiar los métodos de autenticación después de la implementación, es necesario editar el archivo de configuración manualmente.

### <a name="c-based-security-agent"></a>Agente de seguridad basado en C#

Edite _Authentication.config_ con los siguientes parámetros:

```xml
<Authentication>
  <add key="deviceId" value=""/>
  <add key="gatewayHostname" value=""/>
  <add key="filePath" value=""/>
  <add key="type" value=""/>
  <add key="identity" value=""/>
  <add key="certificateLocationKind" value="" />
</Authentication>
```

### <a name="c-based-security-agent"></a>Agente de seguridad basado en C

Edite _LocalConfiguration.json_ con los siguientes parámetros:

```json
"Authentication" : {
    "Identity" : "",
    "AuthenticationMethod" : "",
    "FilePath" : "",
    "DeviceId" : "",
    "HostName" : ""
}
```

## <a name="see-also"></a>Consulte también

- [Información general de los agentes de seguridad](security-agent-architecture.md)
- [Implementación de un agente de seguridad](how-to-deploy-agent.md)
- [Acceso a datos de seguridad sin procesar](how-to-security-data-access.md)
