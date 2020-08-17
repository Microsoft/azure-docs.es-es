---
title: Supervisión de las copias de seguridad con el Explorador de Backup
description: En este artículo se describe cómo usar el Explorador de Backup para realizar la supervisión en tiempo real de las copias de seguridad en almacenes, suscripciones, regiones e inquilinos.
ms.reviewer: dcurwin
ms.topic: conceptual
ms.date: 02/03/2020
ms.openlocfilehash: f514f899f2db70ecb2924c2e627f3bed28e3dcad
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092803"
---
# <a name="monitor-your-backups-with-backup-explorer"></a>Supervisión de las copias de seguridad con el Explorador de Backup

A medida que las organizaciones realizan copias de seguridad de más máquinas en la nube, es cada vez más importante supervisar estas copias de seguridad de forma eficaz. La mejor manera de empezar es usar una ubicación central para ver la información operativa a lo largo de una gran extensión.

El Explorador de Backup es un libro de Azure Monitor integrado que proporciona a los clientes de Azure Backup esta única ubicación central. El Explorador de Backup ayuda a supervisar las actividades operativas en toda la extensión de Backup en Azure, lo que abarca inquilinos, ubicaciones, suscripciones, grupos de recursos y almacenes. En términos generales, el Explorador de Backup ofrece las funcionalidades siguientes:

* **Perspectiva a escala**: Obtenga una vista agregada de los elementos de copia de seguridad, los trabajos, las alertas, las directivas y los recursos no configurados para la copia de seguridad en toda la extensión.
* **Análisis de exploración en profundidad**: Muestre información detallada sobre cada uno de los trabajos, alertas, directivas y elementos de copia de seguridad en un solo lugar.
* **Interfaces accionables**: Después de identificar un problema, puede resolverlo sin problemas en el elemento de copia de seguridad o recurso de Azure correspondientes.

Estas funcionalidades se proporcionan de forma integrada a través de la integración nativa con Azure Resource Graph y los libros de Azure Monitor.

> [!NOTE]
>
> * El Explorador de Backup solo está disponible actualmente para los datos de máquinas virtuales (VM) de Azure.
> * El Explorador de Backup está pensado para servir de panel operativo donde ver información acerca de las copias de seguridad en los últimos 7 días (máximo).
> * El Explorador de Backup no se admite actualmente en las nubes nacionales.
> * Actualmente no se admite la personalización de la plantilla del Explorador de Backup.
> * No se recomienda escribir automatizaciones personalizadas en los datos de Azure Resource Graph.
> * En la actualidad, el Explorador de Backup permite supervisar copias de seguridad en un máximo de 1000 suscripciones (entre inquilinos).

## <a name="get-started"></a>Introducción

Para obtener acceso al Explorador de Backup, puede ir a cualquiera de los almacenes de Recovery Services y hacer clic en el vínculo **Explorador de Backup** en el panel **Información general**.

![Vínculo rápido al almacén](media/backup-azure-monitor-with-backup-explorer/vault-quick-link.png)

Al seleccionar el vínculo, se abre el Explorador de Backup, que proporciona una vista agregada de todos los almacenes y suscripciones a los que tiene acceso. Si usa una cuenta de Azure Lighthouse, puede ver los datos de todos los inquilinos a los que tiene acceso. Para obtener más información, consulte la sección "Vistas entre inquilinos" al final de este artículo.

![Página de aterrizaje del Explorador de Backup](media/backup-azure-monitor-with-backup-explorer/explorer-landing-page.png)

## <a name="backup-explorer-use-cases"></a>Casos de uso del Explorador de Backup

El Explorador de Backup muestra varias pestañas, donde cada una proporciona información detallada sobre un determinado artefacto de copia de seguridad (por ejemplo, un elemento de copia de seguridad, un trabajo o una directiva). En la sección siguiente se proporciona una breve descripción de cada pestaña. Los vídeos proporcionan casos de uso de ejemplo para cada artefacto de copia de seguridad, junto con descripciones de los controles disponibles.

### <a name="the-summary-tab"></a>Pestaña Resumen

La pestaña **Resumen** ofrece una vista rápida de la condición general de la extensión de copia de seguridad. Por ejemplo, puede ver el número de elementos que se protegen, el número de elementos para los que no se ha habilitado la protección o cuántos trabajos se completaron correctamente en las últimas 24 horas.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nQYd]

### <a name="the-backup-items-tab"></a>Pestaña Elementos de copia de seguridad

Puede filtrar y ver cada uno de los elementos de copia de seguridad por suscripción, almacén y otras características. Al seleccionar el nombre de un elemento de copia de seguridad, puede abrir el panel de Azure para ese elemento. Por ejemplo, en la tabla, es posible que observe que se produjo un error en la última copia de seguridad del elemento *X*. Al seleccionar *X*, puede abrir el panel **Copia de seguridad** del elemento, donde puede desencadenar una operación de copia de seguridad a petición.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nQYc]

### <a name="the-jobs-tab"></a>Pestaña Trabajos

Seleccione la pestaña **Trabajos** para ver los detalles de todos los trabajos que se han desencadenado durante los últimos 7 días. Aquí, puede filtrar por *Operación de trabajo*, *Estado de trabajo* y *Código de error* (en el caso de trabajos con errores).

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nOrh]

### <a name="the-alerts-tab"></a>Pestaña Alertas

Seleccione la pestaña **alertas** para ver los detalles de todas las alertas que se han generado en los almacenes durante los últimos 7 días. Puede filtrar las alertas por tipo (*Error de Backup* o *Error de restauración*), el estado actual (*Activo* o *Resuelto*) y la gravedad (*Crítica*, *Advertencia*o *Información*). También puede seleccionar un vínculo para ir a la VM de Azure y realizar las acciones necesarias.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nTxe]

### <a name="the-policies-tab"></a>Pestaña Directivas

Puede seleccionar la pestaña **Directivas** para ver información clave sobre todas las directivas de copia de seguridad que se han creado en la extensión de copias de seguridad. Puede ver el número de elementos asociados a cada directiva, junto con la duración de retención y la frecuencia de copia de seguridad especificadas por la directiva.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nLKV]

### <a name="the-backup-not-enabled-tab"></a>Pestaña Copia de seguridad no habilitada

La copia de seguridad debe estar habilitada para todas las máquinas que requieren protección. Con el Explorador de Backup, los administradores de copias de seguridad pueden identificar rápidamente las máquinas de una organización que aún no están protegidas por copias de seguridad. Para ver esta información, seleccione la pestaña **Copia de seguridad no habilitada**.

En el panel **Copia de seguridad no habilitada** se muestra una tabla con una lista de máquinas sin protección. Su organización puede asignar etiquetas diferentes a las máquinas de producción y las máquinas de prueba, o a las máquinas que atienden diversas funciones. Dado que cada clase de máquinas necesita una directiva de copia de seguridad independiente, el filtrado por etiquetas le ayuda a ver la información específica de cada una. Al seleccionar el nombre de cualquier máquina, se le redirige al panel **Configurar copia de seguridad** de la máquina, donde puede elegir aplicar una directiva de copia de seguridad adecuada.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nQXZ]

## <a name="export-to-excel"></a>Exportación a Excel

Puede exportar el contenido de cualquier tabla o gráfico como hoja de cálculo de Excel. El contenido se exporta tal cual, con los filtros existentes aplicados. Para exportar filas adicionales de una tabla, puede aumentar el número de filas que se muestran en la página mediante la lista desplegable **Filas por página** en la parte superior de cada pestaña.

## <a name="pin-to-the-dashboard"></a>Anclar al panel

Puede seleccionar el icono de anclaje en la parte superior de cada tabla o gráfico para anclarlos a su panel de Azure Portal. Anclar esta información ayuda a crear un panel personalizado que se adapta para mostrar la información más importante para usted.

## <a name="cross-tenant-views"></a>Vistas entre inquilinos

Si es usuario de Azure Lighthouse con acceso delegado a las suscripciones en varios entornos de inquilinos, puede usar el filtro de suscripción predeterminado. Para mostrar las suscripciones para las que desea ver los datos, seleccione el icono de "filtro" en la parte superior derecha de Azure Portal. Cuando se usa esta característica, el Explorador de Backup agrega la información sobre todos los almacenes en las suscripciones seleccionadas. Para más información, consulte [¿Qué es Azure Lighthouse?](../lighthouse/overview.md)

## <a name="next-steps"></a>Pasos siguientes

[Aprenda a usar Azure Monitor para obtener información sobre los datos de copia de seguridad.](./backup-azure-monitoring-use-azuremonitor.md)
