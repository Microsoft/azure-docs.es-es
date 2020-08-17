---
title: Configuración de diagnósticos
titleSuffix: Azure Digital Twins
description: Consulte cómo habilitar el registro con la configuración de diagnóstico.
author: baanders
ms.author: baanders
ms.date: 7/28/2020
ms.topic: troubleshooting
ms.service: digital-twins
ms.openlocfilehash: 402eda92d5cc8e5c2566345d31c9ae625b82695c
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87809412"
---
# <a name="troubleshooting-azure-digital-twins-diagnostics-logging"></a>Solución de problemas de Azure Digital Twins: Registro de diagnóstico

Azure Digital Twins recopila [métricas](troubleshoot-metrics.md) para la instancia del servicio que proporcionan información sobre el estado de los recursos. Puede usar estas métricas para evaluar el estado general del servicio Azure Digital Twins y los recursos conectados a él. Estas estadísticas orientadas al usuario ayudan a ver lo que está ocurriendo con su instancia de Azure Digital Twins y a realizar el análisis de la causa raíz de los problemas sin necesidad de ponerse en contacto con el soporte técnico de Azure.

En este artículo se muestra cómo activar el **registro de diagnóstico** para los datos de métricas de la instancia de Azure Digital Twins. Estos registros pueden ayudarle a solucionar problemas con el servicio.

## <a name="turn-on-diagnostic-settings-with-the-azure-portal"></a>Activación de la configuración de diagnóstico con Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la instancia de Azure Digital Twins. Escriba el nombre en la barra de búsqueda para encontrarla. 

2. En el menú, seleccione **Configuración de diagnóstico** y **Agregar configuración de diagnóstico**.

    :::image type="content" source="media/troubleshoot-metrics/diagnostic-settings.png" alt-text="Captura de pantalla que muestra la página de configuración de diagnóstico y el botón para agregarla":::

3. En la página siguiente, rellene los valores a continuación:
     * **Diagnostic setting name** (Nombre de la configuración de diagnóstico): Asigne un nombre a la configuración de diagnóstico.
     * **Category details** (Detalles de categoría): Elija qué operaciones desea supervisar y marque las casillas para habilitar el diagnóstico para ellas. Las operaciones sobre las que puede informar la configuración de diagnóstico son:
        - DigitalTwinsOperation
        - EventRoutesOperation
        - ModelsOperation
        - QueryOperation
        - AllMetrics
     * **Detalles de destino**: Elija dónde desea enviar los registros. También puede seleccionar cualquier combinación de las tres opciones:
        - Enviar a Log Analytics
        - Archivar en una cuenta de almacenamiento
        - Transmitir a un centro de eventos

        Si es necesario para el destino seleccionado, se le pedirá que rellene detalles adicionales.  
    
4. Guarde la configuración nueva. 

    :::image type="content" source="media/troubleshoot-diagnostics/diagnostic-settings-details.png" alt-text="Captura de pantalla que muestra la página de configuración de diagnóstico y el botón para agregarla":::

La nueva configuración surte efecto en unos 10 minutos. Después, los registros aparecen en el destino configurado en la página **Configuración de diagnóstico** de la instancia. 

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la configuración de diagnóstico, consulte el artículo [*Recopilación y consumo de los datos de registro provenientes de los recursos de Azure*](../azure-monitor/platform/platform-logs-overview.md).
* Para información sobre las métricas de Azure Digital Twins, consulte el artículo [*Solución de problemas: visualización de las métricas con Azure Monitor*](troubleshoot-metrics.md).
