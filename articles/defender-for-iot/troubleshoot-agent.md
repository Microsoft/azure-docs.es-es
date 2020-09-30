---
title: Solución de problemas de inicio del agente de seguridad (Linux)
description: Solución de los problemas de funcionamiento de los agentes de seguridad de Azure Defender para IoT en Linux.
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
ms.openlocfilehash: e9904e9157a560e2a4853a1a9cd37977defe73ea
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931915"
---
# <a name="security-agent-troubleshoot-guide-linux"></a>Guía de solución de problemas de un agente de seguridad (Linux)

En este artículo se explica cómo resolver posibles problemas en el proceso de inicio del agente de seguridad.

El agente de Azure Defender para IoT se inicia automáticamente inmediatamente después de la instalación. El proceso de inicio del agente incluye la lectura de la configuración local, la conexión a Azure IoT Hub y la recuperación de la configuración gemela remota. Si se produce un error en cualquiera de estos pasos, es posible que se produzca un error en el agente de seguridad.

En esta guía de solución de problemas, aprenderá a:

> [!div class="checklist"]
> * Validar si el agente de seguridad se encuentra en ejecución
> * Obtener errores del agente de seguridad
> * Conocer y remediar los errores del agente de seguridad

## <a name="validate-if-the-security-agent-is-running"></a>Validar si el agente de seguridad se encuentra en ejecución

1. Para validar si el agente de seguridad se encuentra en ejecución, espere unos minutos después de instalar el agente y ejecute el siguiente comando.
     <br>

    **Agente de C**

    ```bash
    grep "ASC for IoT Agent initialized" /var/log/syslog
    ```

    **Agente de C#**

    ```bash
    grep "Agent is initialized!" /var/log/syslog
    ```

1. Si el comando devuelve una línea vacía, significa que el agente de seguridad no se pudo iniciar correctamente.

## <a name="force-stop-the-security-agent"></a>Forzar la detención del agente de seguridad

En los casos en los que el agente de seguridad no se puede iniciar, deténgalo con el siguiente comando y, después, continúe con la tabla de errores siguiente:

```bash
systemctl stop ASCIoTAgent.service
```

## <a name="get-security-agent-errors"></a>Obtener errores del agente de seguridad

1. Recupere los errores del agente de seguridad mediante la ejecución del siguiente comando:

    ```bash
    grep ASCIoTAgent /var/log/syslog
    ```

1. El comando de error Get del agente de seguridad recupera todos los registros creados por el agente de Defender para IoT. Use la tabla siguiente para conocer los errores y realizar los pasos correctos para corregirlos.

> [!Note]
> Los registros de errores se muestran en orden cronológico. Asegúrese de anotar la marca de tiempo de cada error para ayudarle a solucionarlo.

## <a name="restart-the-agent"></a>Reinicio del agente

1. Después de localizar y corregir un error del agente de seguridad, intente reiniciar el agente con la ejecución del siguiente comando.

    ```bash
    systemctl restart ASCIoTAgent.service
    ```

1. Repita el proceso anterior para recuperar la detención y recuperar los errores si el agente sigue produciendo algún error en el proceso de inicio.

## <a name="understand-security-agent-errors"></a>Descripción de los errores del agente de seguridad

La mayoría de los errores del agente de seguridad se muestran en el siguiente formato:

```
Defender for IoT agent encountered an error! Error in: {Error Code}, reason: {Error sub code}, extra details: {error specific details}
```

| Código de error | Subcódigo de error | Detalles del error | Corregir en C | Corregir en C# |
|:-----------|:---------------|:--------|:------------|:------------|
| Configuración local | Configuración que falta | Falta una configuración en el archivo de configuración local. El mensaje de error debe indicar la clave que falta. | Agregue la clave que falta al archivo /var/LocalConfiguration.json y consulte [cs-localconfig-reference](azure-iot-security-local-configuration-c.md) para obtener más información.| Agregue la clave que falta al archivo General.config y consulte [c#-localconfig-reference](azure-iot-security-local-configuration-csharp.md) para obtener más información. |
| Configuración local | No se puede analizar la configuración | No se puede analizar un valor de configuración. El mensaje de error debe indicar la clave que no se puede analizar. No se puede analizar un valor de configuración porque el valor no tiene el tipo esperado o está fuera del intervalo. | Corrija el valor de la clave en el archivo /var/LocalConfiguration.json, para que coincida con el esquema LocalConfiguration, y consulte [c#-localconfig-reference](azure-iot-security-local-configuration-csharp.md) para obtener más información. |  Corrija el valor de la clave en el archivo General.config, para que coincida con el esquema, y consulte [cs-localconfig-reference](azure-iot-security-local-configuration-c.md) para obtener más información.|
| Configuración local | Formato de archivo | No se pudo analizar el archivo de configuración. | El archivo de configuración está dañado, descargue el agente y vuelva a instalarlo. | |
| Configuración remota | Tiempo de espera | El agente no pudo capturar el módulo gemelo azureiotsecurity durante el tiempo de espera. | Asegúrese de que la configuración de la autenticación es correcta y vuelva a intentarlo. | El agente no pudo capturar el módulo gemelo azureiotsecurity durante el tiempo de espera. | Asegúrese de que la configuración de la autenticación es correcta y vuelva a intentarlo. |
| Authentication | El archivo no existe | El archivo de la ruta de acceso especificada no existe. | Asegúrese de que el archivo existe en la ruta de acceso especificada o vaya al archivo **LocalConfiguration.json** y cambie la configuración de **FilePath**. | Asegúrese de que el archivo existe en la ruta de acceso especificada o vaya al archivo **Authentication.config** y cambie la configuración de **FilePath**.|
| Authentication | Permiso del archivo | El agente no tiene permisos suficientes para abrir el archivo. | Conceda al usuario **asciotagent** permisos de lectura del archivo de la ruta de acceso especificada. | Asegúrese de que se puede acceder al archivo. |
| Authentication | Formato de archivo | El archivo proporcionado no está en el formato correcto. | Asegúrese de que el archivo está en el formato correcto. Los tipos de archivo compatibles son .pfx y .pem. | Asegúrese de que el archivo es un archivo de certificado válido. |
| Authentication | No autorizado | El agente no pudo autenticarse en IoT Hub con las credenciales proporcionadas. | Valide la configuración de autenticación del archivo LocalConfiguration, repase la configuración de autenticación, asegúrese de que todos los detalles son correctos y compruebe que el secreto del archivo coincide con la identidad autenticada. | Valide la configuración de autenticación del archivo Authentication.config, repase la configuración de autenticación, asegúrese de que todos los detalles son correctos y, después, compruebe que el secreto del archivo coincide con la identidad autenticada.
| Authentication | No encontrado | Se encontró el dispositivo o módulo. | Valide la configuración de autenticación: asegúrese de que el nombre de host es correcto, de que el dispositivo existe en IoT Hub y de que tiene un módulo gemelo azureiotsecurity. |  Valide la configuración de autenticación: asegúrese de que el nombre de host es correcto, de que el dispositivo existe en IoT Hub y de que tiene un módulo gemelo azureiotsecurity. |
| Authentication | Configuración que falta | Falta una configuración en el archivo *Authentication.config*. El mensaje de error debe indicar la clave que falta. | Agregue la clave que falta al archivo *LocalConfiguration.json*.| Agregue la clave que falta al archivo *Authentication.config* y consulte [c#-localconfig-reference](azure-iot-security-local-configuration-csharp.md) para obtener más información. |
| Authentication | No se puede analizar la configuración | No se puede analizar un valor de configuración. El mensaje de error debe indicar la clave que no se puede analizar. No se puede analizar un valor de configuración porque el valor no tiene el tipo esperado o está fuera del intervalo. |Corrija el valor de la clave en el archivo **LocalConfiguration.json**. |Corrija el valor de la clave en el archivo **Authentication.config**, para que coincida con el esquema, y consulte [cs-localconfig-reference](azure-iot-security-local-configuration-c.md) para obtener más información.|
|

## <a name="next-steps"></a>Pasos siguientes

- Lea la [información general](overview.md) del servicio Defender para IoT.
- Obtenga más información sobre la [arquitectura](architecture.md) de Defender para IoT.
- Habilite el [servicio](quickstart-onboard-iot-hub.md) Defender para IoT.
- Lea las [preguntas más frecuentes](resources-frequently-asked-questions.md) del servicio Defender para IoT.
- Aprenda a acceder a [datos de seguridad sin procesar](how-to-security-data-access.md)
- Obtenga información acerca de las [recomendaciones](concept-recommendations.md)
- Obtenga información sobre las [alertas de seguridad](concept-security-alerts.md)
