---
title: Configuración de alertas
titleSuffix: Azure Digital Twins
description: Consulte cómo habilitar las alertas en las métricas de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 7/28/2020
ms.topic: troubleshooting
ms.service: digital-twins
ms.openlocfilehash: ded2f54379e60e8e3fc26d9c2166930a6f953078
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88854855"
---
# <a name="troubleshooting-azure-digital-twins-alerts"></a>Solución de problemas de Azure Digital Twins: Alertas

Azure Digital Twins recopila [métricas](troubleshoot-metrics.md) para la instancia del servicio que proporcionan información sobre el estado de los recursos. Puede usar estas métricas para evaluar el estado general del servicio Azure Digital Twins y los recursos conectados a él.

Las **alertas** le informan de manera proactiva cuando se detectan condiciones importantes en los datos de métricas. Le permiten identificar y solucionar los problemas antes de que los usuarios del sistema puedan verlos. Puede leer más información sobre las alertas en [*Información general sobre las alertas en Microsoft Azure*](../azure-monitor/platform/alerts-overview.md).

## <a name="turn-on-alerts"></a>Activación de las alertas

Aquí se describe cómo habilitar las alertas para la instancia de Azure Digital Twins:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la instancia de Azure Digital Twins. Escriba el nombre en la barra de búsqueda para encontrarla. 

2. En el menú, seleccione **Alertas** y, luego, **+ Nueva regla de alertas**.

3. En la página *Crear regla de alerta* que se muestra a continuación, puede seguir las indicaciones para definir las condiciones, las acciones que se deben desencadenar y los detalles de la alerta.     
    * Los detalles del **ámbito** se deberían rellenar automáticamente con los detalles correspondientes a su instancia.
    * Definirá los detalles de **Condición** y **Grupo de acciones** para personalizar los desencadenadores y las respuestas de alertas.
    * En la sección **Detalles de la regla de alertas**, escriba el _Nombre de la regla de alertas_ y una _Descripción (opcional)_ . Puede seleccionar la casilla _Habilitar la regla tras la creación_ si quiere que la alerta se active en cuanto se cree.
    * En función de algunas condiciones y acciones que agregue, deberá seleccionar _Guardar regla de alerta en el grupo de recursos_ y _Gravedad_ desde las listas desplegables respectivas.

4. Seleccione el botón _Crear regla de alertas_ para crear la regla de alertas.

:::image type="content" source="media/troubleshoot-alerts/create-alert-rule.png" alt-text="Captura de pantalla que muestra la página Crear regla de alerta con secciones para el ámbito, la condición y el grupo de acciones" lightbox="media/troubleshoot-alerts/create-alert-rule.png":::

:::image type="content" source="media/troubleshoot-alerts/alert-rule-details.png" alt-text="Captura de pantalla que muestra la sección de detalles de la regla de alertas de la creación de una regla de alertas" lightbox="media/troubleshoot-alerts/alert-rule-details.png":::

Para un tutorial guiado sobre cómo rellenar estos campos, consulte [*Información general sobre las alertas en Microsoft Azure*](../azure-monitor/platform/alerts-overview.md). A continuación se muestran algunos ejemplos de cómo se verán los pasos en Azure Digital Twins.

Este es un extracto del proceso *Seleccionar condición* que ilustra los tipos de señales de alerta que están disponibles para Azure Digital Twins. En esta página puede filtrar el tipo de señal y seleccionar en una lista la señal que quiere.

:::image type="content" source="media/troubleshoot-alerts/configure-signal-logic.png" alt-text="Captura de pantalla que muestra la primera página Configurar lógica de señal. Aparece resaltado el cuadro Tipo de señal para seleccionar métricas o registros de actividad y una lista de métricas debajo que se puede seleccionar":::

Después de seleccionar una señal, se le pedirá configurar la lógica de la alerta. Puede filtrar en una dimensión, establecer un valor de umbral para la alerta y establecer la frecuencia de las comprobaciones de la condición. Este es un ejemplo de configuración de una alerta para cuando la métrica promedio de Frecuencia de errores de enrutamiento supere el 5 %.

:::image type="content" source="media/troubleshoot-alerts/configure-signal-logic-2.png" alt-text="Captura de pantalla que muestra la segunda página Configurar lógica de señal. La página muestra el historial de métricas, tiene un área para filtrar en una dimensión, como operaciones de Event Grid, y una sección para definir la lógica de alerta, como el promedio es mayor que 5":::
 
Después de configurar las alertas, aparecerán de nuevo en la página *Alertas* correspondiente a la instancia.
 
:::image type="content" source="media/troubleshoot-alerts/alerts-post.png" alt-text="Captura de pantalla que muestra la página Alertas y el botón para agregar. Hay una alerta configurada" lightbox="media/troubleshoot-alerts/alerts-post.png":::


## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre las alertas con Azure Monitor, consulte [*Información general sobre las alertas en Microsoft Azure*](../azure-monitor/platform/alerts-overview.md).
* Para información sobre las métricas de Azure Digital Twins, consulte el artículo [*Solución de problemas: visualización de las métricas con Azure Monitor*](troubleshoot-metrics.md).
* Para ver cómo habilitar el registro de diagnóstico para las métricas, consulte [*Solución de problemas: Configuración de diagnósticos*](troubleshoot-diagnostics.md).
