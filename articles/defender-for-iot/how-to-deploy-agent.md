---
title: Selección e implementación de agentes de seguridad
description: Aprenda a seleccionar e implementar agentes de seguridad de Defender for IoT en dispositivos IoT.
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
ms.date: 07/23/2019
ms.author: mlottner
ms.openlocfilehash: 8e18b79cc14fe98879ec97361f6e275d8fd918bb
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932652"
---
# <a name="select-and-deploy-a-security-agent-on-your-iot-device"></a>Seleccionar e implementar un agente de seguridad en el dispositivo IoT

Defender for IoT proporciona arquitecturas de referencia para los agentes de seguridad que supervisan y recopilan datos de dispositivos IoT.
Para más información, vea [Arquitectura de referencia del agente de seguridad](security-agent-architecture.md).

Los agentes de seguridad se desarrollan como proyectos de código abierto y están disponibles en dos tipos: <br> [C](https://aka.ms/iot-security-github-c) y [C#](https://aka.ms/iot-security-github-cs).

En este artículo aprenderá a:

> [!div class="checklist"]
> * Comparar tipos de agente de seguridad
> * Descubrir las plataformas de agente admitidas
> * Elegir el tipo de agente correcto para su solución

## <a name="understand-security-agent-options"></a>Descripción de las opciones de agente de seguridad

Cada tipo de agente de seguridad de Defender for IoT ofrece el mismo conjunto de características y admite opciones de configuración similares.

El agente de seguridad basado en C tiene una superficie de memoria menor y es la opción ideal en dispositivos con pocos recursos disponibles.

|     | Agente de seguridad basado en C | Agente de seguridad basado en C# |
| --- | ----------- | --------- |
| **Código abierto** | Disponible con [licencia de MIT](https://en.wikipedia.org/wiki/MIT_License) en [GitHub](https://aka.ms/iot-security-github-c) | Disponible con [licencia de MIT](https://en.wikipedia.org/wiki/MIT_License) en [GitHub](https://aka.ms/iot-security-github-cs) |
| **Lenguaje de desarrollo**    | C | C# |
| **¿Compatible con plataformas Windows?** | No | Sí |
| **Requisitos previos de Windows** | --- | [WMI](https://docs.microsoft.com/windows/desktop/wmisdk/) |
| **¿Compatible con plataformas Linux?** | Sí, x64 y x86 | Sí, solo x64 |
| **Requisitos previos de Linux** | libunwind8, libcurl3, uuid-runtime, auditd, audispd-plugins | libunwind8, libcurl3, uuid-runtime, auditd, audispd-plugins, sudo, netstat, iptables |
| **Superficie de disco** | 10,5 MB | 90 MB |
| **Superficie de memoria (promedio)** | 5,5 MB | 33 MB |
| **[Autenticación](concept-security-agent-authentication-methods.md) en IoT Hub** | Sí | Sí |
| **[Recopilación](how-to-agent-configuration.md#supported-security-events) de datos de seguridad** | Sí | Sí |
| **Agregación de eventos** | Sí | Sí |
| **Configuración remota por medio de [módulo gemelo de seguridad](concept-security-module.md)** | Sí | Sí |

## <a name="security-agent-installation-guidelines"></a>Directrices de instalación del agente de seguridad

En **Windows**: El script Install SecurityAgent.ps1 debe ejecutarse desde una ventana de administrador de PowerShell.

En **Linux**: InstallSecurityAgent.sh debe ejecutarse como superusuario. Se recomienda colocar "sudo" como prefijo del comando de instalación.

## <a name="choose-an-agent-flavor"></a>Elegir un tipo de agente

Responda las siguientes preguntas sobre sus dispositivos IoT para seleccionar al agente correcto:

- ¿Usa _Windows Server_ o _Windows IoT Core_?

    [Implementación de un agente de seguridad basado en C# para Windows](how-to-deploy-windows-cs.md)

- ¿Usa una distribución Linux con arquitectura x86?

    [Implementación de un agente de seguridad basado en C para Linux](how-to-deploy-linux-c.md)

- ¿Usa una distribución Linux con arquitectura x64?

    Se pueden usar ambos tipos de agente. <br>
    [Implementación de un agente de seguridad basado en C para Linux](how-to-deploy-linux-c.md) y/o [Implementación de un agente de seguridad basado en C# para Linux](how-to-deploy-linux-cs.md)

Ambos tipos de agente ofrecen el mismo conjunto de características y admiten opciones de configuración similares.
Vea [Comparación de agentes de seguridad](how-to-deploy-agent.md#understand-security-agent-options) para obtener más información.

## <a name="supported-platforms"></a>Plataformas compatibles

La siguiente lista incluye todas las plataformas admitidas actualmente.

|Agente de Defender for IoT |Sistema operativo |Architecture |
|--------------|------------|--------------|
|C|Ubuntu 16.04 |    x64|
|C|Ubuntu 18.04 |    x64, ARMv7|
|C|Debian 9 |    x64, x86|
|C#|Ubuntu 16.04     |x64|
|C#|Ubuntu 18.04    |x64, ARMv7|
|C#|Debian 9    |x64|
|C#|Windows Server 2016|    X64|
|C#|Windows 10 IoT Core, compilación 17763    |x64|
|

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las opciones de configuración, prosiga con la guía de procedimientos para configurar el agente.
> [!div class="nextstepaction"]
> [Tutorial: Configurar agentes de seguridad](./how-to-agent-configuration.md)
