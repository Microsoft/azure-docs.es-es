---
title: Validación de alertas (archivo de prueba EICAR) en Azure Security Center | Microsoft Docs
description: Este documento le ayuda a validar las alertas de seguridad en Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: f8f17a55-e672-4d86-8ba9-6c3ce2e71a57
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/04/2019
ms.author: memildin
ms.openlocfilehash: cf73b3949b0a0dc1e76ebdebb191af0a33ce22ff
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180480"
---
# <a name="alert-validation-in-azure-security-center"></a>Validación de alertas en Azure Security Center
Este documento le ayuda a comprobar si el sistema está configurado correctamente para las alertas de Azure Security Center.

## <a name="what-are-security-alerts"></a>¿Qué son las alertas de seguridad?
Las alertas son notificaciones que Security Center genera cuando detecta amenazas en los recursos. Asigna prioridades y enumera las alertas, junto con la información necesaria para que pueda investigar rápidamente el problema. Security Center también proporciona recomendaciones sobre el modo en que puede corregir un ataque.
Para obtener más información, consulte [Alertas de seguridad en Azure Security Center](security-center-alerts-overview.md) y [Administración y respuesta a las alertas de seguridad](security-center-managing-and-responding-alerts.md).

## <a name="alert-validation"></a>Validación de alertas

* [Windows](#validate-windows)
* [Linux](#validate-linux)
* [Kubernetes](#validate-kubernetes)

## <a name="validate-alerts-on-windows-vms"></a>Validación de alertas en máquinas virtuales Windows <a name="validate-windows"></a>

Una vez que el agente de Security Center esté instalado en el equipo, siga estos pasos desde el equipo donde desee que esté el recurso atacado de la alerta:

1. Copie un archivo ejecutable (por ejemplo **calc.exe**) en el escritorio del equipo o en otro directorio que prefiera y cambie el nombre a **ASC_AlertTest_662jfi039N.exe**.
1. Abra el símbolo del sistema y ejecute este archivo con un argumento (un nombre de argumento falso), por ejemplo, ```ASC_AlertTest_662jfi039N.exe -foo```
1. Espere de cinco a diez minutos y abra Alertas de Security Center. Debería aparecer una alerta.

> [!NOTE]
> Al revisar esta alerta de prueba para Windows, asegúrese de que el campo **Arguments Auditing Enabled** (Auditoría de argumentos habilitada) aparece como **true**. Si es **false**, debe habilitar la auditoría de argumentos de línea de comandos. Para habilitarlo, utilice la línea de comandos siguiente:
>
>```reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\system\Audit" /f /v "ProcessCreationIncludeCmdLine_Enabled"```

## <a name="validate-alerts-on-linux-vms"></a>Validación de alertas en máquinas virtuales Linux <a name="validate-linux"></a>

Una vez que el agente de Security Center esté instalado en el equipo, siga estos pasos desde el equipo donde desee que esté el recurso atacado de la alerta:
1. Copie un archivo ejecutable en una ubicación adecuada y cambie el nombre a **./asc_alerttest_662jfi039n**, por ejemplo:

    ```cp /bin/echo ./asc_alerttest_662jfi039n```

1. Abra el símbolo del sistema y ejecute este archivo:

    ```./asc_alerttest_662jfi039n testing eicar pipe```

1. Espere de cinco a diez minutos y abra Alertas de Security Center. Debería aparecer una alerta.


## <a name="validate-alerts-on-kubernetes"></a>Validación de alertas en Kubernetes <a name="validate-kubernetes"></a>

Si usa la característica de vista previa de Security Center referentes a la integración de Azure Kubernetes Service, ejecute el siguiente comando kubectl para comprobar que las alertas funcionan:

```kubectl get pods --namespace=asc-alerttest-662jfi039n```

Para obtener más información sobre la integración de Azure Kubernetes Service y Azure Security Center, consulte [este artículo](azure-kubernetes-service-integration.md).

## <a name="next-steps"></a>Pasos siguientes
En este artículo se explicó el proceso de validación de las alertas. Ahora que conoce esta validación, intente los siguientes pasos:

* [Validación de la detección de amenazas de Azure Key Vault en Azure Security Center](https://techcommunity.microsoft.com/t5/azure-security-center/validating-azure-key-vault-threat-detection-in-azure-security/ba-p/1220336)
* [Administración y respuesta a las alertas de seguridad en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts): obtenga información sobre cómo administrar y responder a los incidentes de seguridad en Security Center.
* [Supervisión del estado de seguridad en Azure Security Center](security-center-monitoring.md): obtenga información sobre cómo supervisar el mantenimiento de los recursos de Azure.
* [Comprender las alertas de seguridad en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-alerts-type): obtenga información sobre los distintos tipos de alertas de seguridad.
