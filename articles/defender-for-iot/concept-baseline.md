---
title: Comprobaciones personalizadas y de línea de base
description: Obtenga información sobre el concepto de línea de base de Azure Defender para IoT.
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
ms.date: 10/07/2019
ms.author: mlottner
ms.openlocfilehash: d97fa4c3c57f6f0dcc5c55b76d839308156c40fb
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932312"
---
# <a name="azure-defender-for-iot-baseline-and-custom-checks"></a>Comprobaciones personalizadas y línea de base de Azure Defender para IoT

En este artículo se explica la línea de base de Defender para IoT y se resumen todas las propiedades asociadas de las comprobaciones personalizadas de línea de base.

## <a name="baseline"></a>Línea base

Una línea de base establece el comportamiento estándar para cada dispositivo y facilita el establecimiento de comportamiento inusual o la desviación de las normas esperadas.

## <a name="baseline-custom-checks"></a>Comprobaciones personalizadas de línea de base

Las comprobaciones personalizadas de base de referencia establecen una lista de comprobaciones personalizada para cada base de referencia del dispositivo mediante **Identidad de módulo gemela** del dispositivo.

## <a name="setting-baseline-properties"></a>Establecimiento de propiedades de línea de base

1. En su IoT Hub, busque y seleccione el dispositivo que quiera cambiar.
1. Haga clic en el dispositivo y, después, en el módulo **azureiotsecurity**.
1. Haga clic en **Identidad de módulo gemela**.
1. Cargue el archivo de **comprobaciones personalizadas de línea de base** en el dispositivo.
1. Agregue propiedades de línea de base al módulo de seguridad y haga clic en **Guardar**.

### <a name="baseline-custom-check-file-example"></a>Ejemplo de archivo de comprobación personalizada de línea de base

Para configurar las comprobaciones personalizadas de línea de base:

   ```json
    "desired": {
      "ms_iotn:urn_azureiot_Security_SecurityAgentConfiguration": {
        "baselineCustomChecksEnabled": {
          "value" : true
        },
        "baselineCustomChecksFilePath": {
          "value" : "/home/user/full_path.xml"
        },
        "baselineCustomChecksFileHash": {
          "value" : "#hashexample!"
        }
      }
    },
   ```

## <a name="baseline-custom-check-properties"></a>Propiedades de comprobación personalizada de línea de base

| Nombre| Status | Valores válidos| Valores predeterminados| Descripción |
|----------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|
|baselineCustomChecksEnabled|Obligatorio: true |Valores válidos:  **Boolean** |Valor predeterminado: **false** |Intervalo de tiempo máximo antes de que los mensajes de prioridad alta se envíen.|
|baselineCustomChecksFilePath |Obligatorio: true|Valores válidos:  **String**, **NULL** |Valor predeterminado: **null** |Ruta de acceso completa de la configuración de XML de línea de base|
|baselineCustomChecksFileHash |Obligatorio: true|Valores válidos:  **String**, **NULL** |Valor predeterminado: **null** |`sha256sum` del archivo de configuración XML. Use la [referencia sha256sum](https://linux.die.net/man/1/sha256sum) para obtener información adicional. |

Para revisar ejemplos de línea de base adicionales, consulte el [ejemplo de línea de base personalizado 1](https://ascforiot.blob.core.windows.net/public/custom_baseline_example_hyperv_ubuntu1804.xml) y el [ejemplo de línea de base personalizado 2](https://ascforiot.blob.core.windows.net/public/oms_audits.xml).

## <a name="next-steps"></a>Pasos siguientes

- Acceso a los [datos de seguridad sin procesar](how-to-security-data-access.md)
- [Investigación de un dispositivo](how-to-investigate-device.md)
- Conozca y explore las [recomendaciones de seguridad](concept-recommendations.md)
- Conozca y explore las [alertas de seguridad](concept-security-alerts.md)
