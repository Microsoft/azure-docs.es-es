---
title: Supervisión de una cuenta de Azure Storage en Azure Portal | Microsoft Docs
description: Aprenda a supervisar una cuenta de almacenamiento en Azure usando Azure Portal y Azure Storage Analytics.
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 01/09/2020
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring
ms.openlocfilehash: 4c78938e2f70342822a4009ed9195196ca486fe0
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88034675"
---
# <a name="monitor-a-storage-account-in-the-azure-portal"></a>Supervisión de una cuenta de almacenamiento en el Portal de Azure

[Análisis de Azure Storage](storage-analytics.md) proporciona métricas para todos los servicios de almacenamiento y registros para blobs, colas y tablas. Puede usar [Azure Portal](https://portal.azure.com) para configurar qué registros y métricas se registran para su cuenta y configurar gráficos que proporcionen representaciones visuales de los datos de las métricas. 

Se recomienda consultar [Azure Monitor para Storage](../../azure-monitor/insights/storage-insights-overview.md) (versión preliminar). Es una característica de Azure Monitor que ofrece una supervisión completa de las cuentas de Azure Storage, ya que ofrece una vista unificada del rendimiento, la capacidad y la disponibilidad de los servicios de Azure Storage. No es preciso habilitar o configurar nada, y puede ver inmediatamente estas métricas desde los gráficos interactivos predefinidos y otras visualizaciones incluidas.

> [!NOTE]
> Existen costos asociados con el análisis de los datos de supervisión en Azure Portal. Para más información, consulte [Storage Analytics](storage-analytics.md).
>
> Azure Files admite actualmente las métricas de Storage Analytics, pero aún no admite el registro.
>
> Las cuentas de almacenamiento de blobs en bloques de rendimiento Premium no admiten las métricas de análisis de almacenamiento, pero admiten el registro, que se puede habilitar mediante programación a través de la API REST o de la biblioteca cliente. Si desea ver métricas con cuentas de almacenamiento de blobs en bloques de rendimiento Premium, considere la posibilidad de usar [Métricas de Azure Storage en Azure Monitor](storage-metrics-in-azure-monitor.md).
>
> Para obtener orientación exhaustiva sobre el uso de análisis de almacenamiento y otras herramientas para identificar, diagnosticar y solucionar problemas relacionados con Azure Storage, consulte [Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage](storage-monitoring-diagnosing-troubleshooting.md).
>

## <a name="configure-monitoring-for-a-storage-account"></a>la supervisión para una cuenta de almacenamiento

1. En [Azure Portal](https://portal.azure.com), seleccione **Cuentas de almacenamiento**y, después, seleccione el nombre de la cuenta de almacenamiento para abrir el panel de esta.
1. En la sección **SUPERVISIÓN** de la hoja de menú, seleccione **Diagnósticos**.

    ![OpcionesSupervisión](./media/storage-monitor-storage-account/storage-enable-metrics-00.png)

1. Seleccione el **tipo** de datos de métricas para cada **servicio** que desee supervisar y la **directiva de retención** para los datos. También puede deshabilitar la supervisión estableciendo el **estado** en **Desactivado**.

    ![OpcionesSupervisión](./media/storage-monitor-storage-account/storage-enable-metrics-01.png)

   Para configurar la directiva de retención de datos, mueva el control deslizante **Retención (en días)** o escriba el número de días que se deben retener los datos, entre 1 y 365. El valor predeterminado para las nuevas cuentas de almacenamiento es de siete días. Si no desea configurar una directiva de retención, escriba un cero. Si no existe una directiva de retención, es posible eliminar los datos de supervisión.

   > [!WARNING]
   > Se le cobrará si elimina manualmente datos de métricas. El sistema eliminará los datos de análisis obsoletos (aquellos anteriores a la directiva de retención de datos) sin costo alguno. Es recomendable configurar una directiva de retención basada en el tiempo que desee retener los datos de análisis de almacenamiento de su cuenta. Vea [Facturación según las métricas de almacenamiento](storage-analytics-metrics.md#billing-on-storage-metrics) para más información.
   >

1. Al finalizar la configuración de la supervisión, seleccione **Guardar**.

Se muestra un conjunto predeterminado de métricas en los gráficos de la hoja de la cuenta de almacenamiento, así como las hojas de cada servicio (Blob, Cola, Tabla y Archivo). Una vez que ha habilitado las métricas de un servicio, puede tardar una hora hasta que los datos aparezcan en los gráficos. Puede seleccionar **Editar** en cualquier gráfico de métricas para configurar qué métricas se muestran en el gráfico.

Puede deshabilitar la recopilación y el registro de métricas estableciendo el **estado** en **Desactivado**.

> [!NOTE]
> Azure Storage usa [Table Storage](storage-introduction.md#table-storage) para almacenar las métricas para la cuenta de almacenamiento y almacena las métricas en tablas en su cuenta. Para obtener más información, vea: [Cómo se almacenan las métricas](storage-analytics-metrics.md#how-metrics-are-stored).
>

## <a name="customize-metrics-charts"></a>Personalización de gráficos de métricas

Utilice el procedimiento siguiente para elegir las métricas de almacenamiento que desea ver en un gráfico de métricas.

1. Comience por mostrar un gráfico de métricas de almacenamiento en Azure Portal. Puede encontrar los gráficos en la **hoja de la cuenta de almacenamiento** y en la hoja **Métricas** de un servicio individual (Blob, Cola, Tabla, Archivo).

   En este ejemplo, se usa el siguiente gráfico que aparece en la **hoja de la cuenta de almacenamiento**:

   ![Selección de gráfico en Azure Portal](./media/storage-monitor-storage-account/stg-customize-chart-00.png)

1. Haga clic en cualquier lugar dentro del gráfico para modificarlo.

1. Después, seleccione el **Intervalo de tiempo** de la métrica para mostrar en el gráfico y el **servicio** (blob, cola, tabla, archivo) cuya métrica desee mostrar. Aquí se seleccionan las métricas de la semana anterior para mostrar para el servicio Blob:

   ![Selección de intervalo de tiempo y servicio en la hoja Editar gráfico](./media/storage-monitor-storage-account/storage-edit-metric-time-range.png)

1. Seleccione las **métricas** individuales que desea que aparezcan en el gráfico y, a continuación, haga clic en **Aceptar**.

   ![Selección de métricas individuales en la hoja Editar gráfico](./media/storage-monitor-storage-account/storage-edit-metric-selections.png)

Estas configuraciones del gráfico no afectan a la recopilación, la incorporación o el almacenamiento de los datos de supervisión en la cuenta de almacenamiento.

### <a name="metrics-availability-in-charts"></a>Disponibilidad de las métricas en los gráficos

La lista de las métricas disponibles cambia en función de qué servicio haya elegido en la lista desplegable y del tipo de unidad del gráfico que está editando. Por ejemplo, puede seleccionar métricas de porcentaje como *PercentNetworkError* y *PercentThrottlingError* solo si está editando un gráfico que muestra las unidades en porcentajes:

![Solicitud de gráfico de porcentaje de errores en Azure Portal](./media/storage-monitor-storage-account/stg-customize-chart-04.png)

### <a name="metrics-resolution"></a>Resolución de métricas

Las métricas seleccionadas en **Diagnóstico** determinan la resolución de las métricas que están disponibles para su cuenta:

* **Agregados**: la supervisión proporciona métricas como entrada/salida, disponibilidad, latencia y porcentajes de éxito. Estas métricas se agregan a los servicios Blob, Cola, Tabla y Archivo.
* **Por API** proporciona una resolución más precisa, con métricas disponibles para operaciones de almacenamiento individuales, además de los agregados de nivel de servicio.

## <a name="configure-metrics-alerts"></a>Configuración de las alertas de métricas

Puede crear alertas para recibir una notificación cuando se alcancen los umbrales para las métricas de los recursos de almacenamiento.

1. Para abrir la **hoja Reglas de alerta**, desplácese hacia abajo hasta la sección **SUPERVISIÓN** de la **hoja Menú** y seleccione **Alertas (clásico)** .
2. Seleccione **Agregar una alerta de métrica (clásica)** para abrir la hoja **Agregar una regla de alerta**.
3. Escriba un **Nombre** y una **Descripción** para la nueva regla de alerta.
4. Seleccione la **métrica** para la que desea agregar una alerta, una **condición** de alerta y un **umbral** de alerta. El tipo de unidad de umbral cambiará en función de la métrica que haya elegido. Por ejemplo, "número" es el tipo de unidad para *ContainerCount*, mientras que la unidad para la métrica *PercentNetworkError* es un porcentaje.
5. Seleccione el **período**. Las métricas que alcancen o superen el umbral dentro de ese período de tiempo desencadenarán una alerta.
6. (Opcional) Configure las notificaciones de **Correo electrónico** y **Webhook**. Para más información sobre webhooks, consulte [Configuración de un webhook en una alerta de métrica de Azure](../../azure-monitor/platform/alerts-webhooks.md). Si no configura las notificaciones de correo electrónico o webhook, las alertas solo aparecerán en Azure Portal.

![Hoja "Agregar una regla de alerta" en Azure Portal](./media/storage-monitor-storage-account/add-alert-rule.png)

## <a name="add-metrics-charts-to-the-portal-dashboard"></a>Adición de gráficos de métricas al panel del portal

Puede agregar gráficos de métricas de Azure Storage a cualquiera de las cuentas de almacenamiento del panel del portal.

1. Seleccione **Editar panel** mientras lo ve en [Azure Portal](https://portal.azure.com).
1. En la **galería de iconos**, seleccione **Buscar iconos por** > **Tipo**.
1. Seleccione **Tipo** > **Cuentas de almacenamiento**.
1. En **Recursos**, seleccione la cuenta de almacenamiento cuyas métricas desea agregar al panel.
1. Seleccione **Categorías** > **Supervisión**.
1. Arrastre y coloque el icono del gráfico en el panel de la métrica que desea mostrar. Repita este paso para todas las métricas que desee que se muestren en el panel. En la siguiente imagen, se resalta el gráfico "Blobs: Número total de solicitudes" como un ejemplo, pero todos los gráficos están disponibles para su ubicación en el panel.

   ![Galería de iconos de Azure Portal](./media/storage-monitor-storage-account/storage-customize-dashboard.png)
1. Seleccione la opción **Personalización finalizada** situada cerca de la parte superior del panel cuando haya terminado de agregar los gráficos.

Una vez que haya agregado los gráficos al panel, los puede personalizar aún más tal y como se describe en Personalización de gráficos de métricas.

## <a name="configure-logging"></a>registro

También puede indicar a Azure Storage que guarde los registros de diagnóstico de lectura, escritura y eliminación de las solicitudes de los servicios Blob, Tabla y Cola. La directiva de retención de datos que estableció también se aplica a estos registros.

> [!NOTE]
> Azure Files admite actualmente las métricas de Storage Analytics, pero aún no admite el registro.
>

1. En [Azure Portal](https://portal.azure.com), seleccione **Cuentas de almacenamiento**y, después, seleccione el nombre de la cuenta de almacenamiento para abrir la hoja de esta.
1. En la sección **Supervisión (clásica)** de la hoja de menú, seleccione **Configuración de diagnóstico (clásica)** .

    ![Elemento de menú Diagnóstico de la sección SUPERVISIÓN de Azure Portal.](./media/storage-monitor-storage-account/storage-enable-metrics-00.png)

1. Asegúrese de que el **estado** está establecido en **Activado** y seleccione los **servicios** para los que desea habilitar el registro.

    ![Configure el registro en Azure Portal.](./media/storage-monitor-storage-account/enable-diagnostics.png)
1. Haga clic en **Save**(Guardar).

Los registros de diagnóstico se guardan en un contenedor de blobs denominado *$logs* en su cuenta de almacenamiento. Para ver los datos de registro se puede usar un explorador de almacenamiento como el [Explorador de Microsoft Azure Storage](https://storageexplorer.com), o bien, mediante programación, la biblioteca de cliente de almacenamiento o PowerShell.

Para más información sobre el acceso al contenedor $logs, vea [Registro de Storage Analytics](storage-analytics-logging.md).

## <a name="next-steps"></a>Pasos siguientes

* Consiga más información acerca de [métricas, registro y facturación](storage-analytics.md) para Storage Analytics.
