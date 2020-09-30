---
title: Supervisión de los servicios de Azure Storage con Azure Monitor para Storage | Microsoft Docs
description: En este artículo se describe la característica Azure Monitor para Storage que proporciona a los administradores de almacenamiento una comprensión rápida de los problemas de uso y rendimiento de sus cuentas de Azure Storage.
ms.subservice: ''
ms.topic: conceptual
author: mrbullwinkle
ms.author: mbullwin
ms.date: 05/11/2020
ms.openlocfilehash: 6d6754594a321d297d983f9bd7e90ce074857919
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961982"
---
# <a name="monitoring-your-storage-service-with-azure-monitor-for-storage"></a>Supervisión del servicio de almacenamiento con Azure Monitor para Storage

Azure Monitor para Storage proporciona una supervisión completa de las cuentas de Azure Storage al ofrecer una vista unificada del rendimiento, la capacidad y la disponibilidad de los servicios de Azure Storage. Es posible observar la capacidad de almacenamiento y el rendimiento de dos maneras: se pueden ver directamente desde una cuenta de almacenamiento, o bien en Azure Monitor para comparar estos valores entre grupos de cuentas de almacenamiento. 

Este artículo le ayudará a comprender la experiencia que proporciona Azure Monitor para Storage para extraer conocimientos prácticos sobre el estado y el rendimiento de las cuentas de almacenamiento a escala, con una funcionalidad para centrarse en las zonas activas y diagnosticar los problemas de latencia, limitación y disponibilidad.

## <a name="introduction-to-azure-monitor-for-storage"></a>Introducción a Azure Monitor para Storage

Antes de profundizar en la experiencia, debe entender cómo se presenta y se visualiza la información. Tanto si selecciona la característica de almacenamiento directamente desde una cuenta de almacenamiento o desde Azure Monitor, Azure Monitor para Storage presenta una experiencia coherente. 

Cuando se combinan, ofrece lo siguiente:

* **Perspectiva a escala**, que muestra una instantánea de la disponibilidad según el estado del servicio de almacenamiento o la operación de API; del uso, con el número total de solicitudes que recibe el servicio de almacenamiento; y de la latencia, que muestra el tiempo medio que tarda el servicio de almacenamiento o el tipo de operación de API en procesar las solicitudes. También puede ver la capacidad por blob, archivo, tabla y cola.

* **Análisis profundo** de una cuenta de almacenamiento concreta para ayudar a diagnosticar problemas o realizar análisis detallados por categoría: disponibilidad, rendimiento, errores y capacidad. Al seleccionar cualquiera de estas opciones, se proporciona una vista detallada de las métricas.  

* **Personalización**, que le permite cambiar las métricas que desea ver y modificar, o establecer umbrales en consonancia con sus límites y guardarlos en su propio libro. Los gráficos del libro se pueden anclar al panel de Azure.  

Esta característica no requiere que se habilite ni configure nada; las métricas de almacenamiento de las cuentas de almacenamiento se recopilan de forma predeterminada. Si no está familiarizado con las métricas disponibles en Azure Storage, revise la descripción y la definición de las métricas de Azure Storage en [Métricas de Azure Storage](../../storage/common/monitor-storage.md).

>[!NOTE]
>El acceso a esta característica no se cobra, solo paga por las características básicas de Azure Monitor esenciales que configure o habilite, como se describe en la página de detalles de [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

>[!NOTE]
>Azure Monitor para Storage no admite [cuentas de uso general v1](../../storage/common/storage-account-overview.md#general-purpose-v1-accounts).
>

## <a name="view-from-azure-monitor"></a>Vista desde Azure Monitor

En Azure Monitor, puede ver los detalles de transacciones, latencia y capacidad de varias cuentas de almacenamiento de su suscripción, así como ayudar a identificar problemas y errores de rendimiento y capacidad.

Para ver el uso y la disponibilidad de las cuentas de almacenamiento en todas las suscripciones, realice los pasos siguientes.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Seleccione **Supervisión** en el panel izquierdo de Azure Portal y, en la sección **Información detallada**, elija **Cuentas de almacenamiento**.

    ![Vista de varias cuentas de almacenamiento](./media/storage-insights-overview/multiple-storage-accounts-view-01.png)

### <a name="overview-workbook"></a>Libro de información general

En el libro **Información general** de la suscripción seleccionada, la tabla muestra las métricas de almacenamiento interactivas y el estado de disponibilidad del servicio para un máximo de 5 cuentas de almacenamiento agrupadas en la suscripción. Puede filtrar los resultados en función de las opciones que seleccione en las siguientes listas desplegables:

* **Subscriptions** (Suscripciones): solo se muestran las suscripciones que tienen cuentas de almacenamiento.  

* **Cuentas de almacenamiento**: de forma predeterminada, se seleccionan previamente 5 cuentas de almacenamiento. Si selecciona todas o varias cuentas de almacenamiento en el selector de ámbito, se devolverán como máximo 200 cuentas de almacenamiento. Por ejemplo, si tiene en total 573 cuentas de almacenamiento entre tres suscripciones que ha seleccionado, solo se mostrarán 200. 

* **Time Range** (Intervalo de tiempo): de forma predeterminada, muestra las últimas cuatro horas de información en función de las selecciones correspondientes realizadas.

El icono de contador bajo las listas desplegables acumula el número total de cuentas de almacenamiento de la suscripción y refleja cuántas se seleccionan del total. La codificación de color condicional o mapas térmicos de las columnas del libro notifican los errores o las métricas de transacción. El color más profundo tiene el valor más alto y un color más claro indica los valores más bajos. En las columnas de errores el valor está en rojo y en las columnas de métricas en azul.

Al seleccionar un valor en las columnas **Availability** (Disponibilidad), **E2E Latency** (Latencia de E2E), **Server Latency** (Latencia del servidor) y **transaction error type/Errors** (tipo de error de transacción/errores), se le dirige a un informe adaptado al tipo específico de métrica de almacenamiento que coincide con la columna seleccionada para esa cuenta de almacenamiento. Para más información sobre los libros de cada categoría, consulte a continuación la sección [Libros de almacenamiento detallados](#detailed-storage-workbooks). 

>[!NOTE]
>Para más información sobre los errores que se pueden mostrar en el informe, consulte [Esquema de tipo de respuesta](../../storage/common/monitor-storage-reference.md#metrics-dimensions) y busque tipos de respuesta, como **ServerOtherError**, **ClientOtherError** y **ClientThrottlingError**. En función de las cuentas de almacenamiento seleccionadas, si se notifican más de tres tipos de errores, todos los demás errores se representan en la categoría **Other** (Otros).

El umbral predeterminado de **Availability** (Disponibilidad) es:

* Warning (Advertencia): 99 %
* Critical (Crítico): 90 %

Para establecer un umbral de disponibilidad en función de los resultados de la observación o de los requisitos, revise [Modificación del umbral de disponibilidad](#modify-the-availability-threshold). 

### <a name="capacity-workbook"></a>Libro de capacidad

Seleccione **Capacity** (Capacidad) en la parte superior de la página; se abre el libro **Capacity** (Capacidad). En él se muestra la cantidad de almacenamiento total usado en la cuenta y la capacidad que usa cada servicio de datos de la cuenta, lo que ayuda a identificar el almacenamiento utilizado en exceso y el infrautilizado.

![Libro de capacidad de varias cuentas de almacenamiento](./media/storage-insights-overview/storage-account-capacity-02.png) 

La codificación de color condicional o mapas térmicos de las columnas del libro notifican las métricas de capacidad con un valor azul. El color más profundo tiene el valor más alto y un color más claro indica los valores más bajos.

Al seleccionar un valor en cualquiera de las columnas del libro, se profundiza en el libro **Capacity** (Capacidad) de la cuenta de almacenamiento. Puede encontrar más información sobre el informe en profundidad en la sección [Libros de almacenamiento detallados](#detailed-storage-workbooks). 

## <a name="view-from-a-storage-account"></a>Vista desde una cuenta de almacenamiento

Para acceder a Azure Monitor para VM directamente desde una máquina virtual, siga estos pasos:

1. En Azure Portal, seleccione Cuentas de almacenamiento.

2. En la lista, elija una cuenta de almacenamiento. En la sección Supervisión, seleccione Información detallada.

    ![Página de información general de la cuenta de almacenamiento seleccionada](./media/storage-insights-overview/storage-account-direct-overview-01.png)

En el libro **Overview** (Información general) de la cuenta de almacenamiento, se muestran varias métricas de rendimiento del almacenamiento que le ayudan evaluar rápidamente:

* El estado del servicio de almacenamiento para ver inmediatamente si hay algún problema que escapa de su control que afecta al servicio Storage en la región en la que está implementado. Esto se indica en la columna **Summary** (Resumen).

* Los gráficos de rendimiento interactivos que muestran los detalles más esenciales relacionados con la capacidad de almacenamiento, la disponibilidad, las transacciones y la latencia.  

* Los iconos de métricas y estado que resaltan la disponibilidad del servicio, el recuento total de transacciones con el servicio de almacenamiento, la latencia de E2E y la latencia del servidor.

Al seleccionar cualquiera de los botones de **Errores** (Failures), **Rendimiento** (Performance), **Availability** (Disponibilidad) y **Capacity** (Capacidad) se abre el libro correspondiente. 

![Página de información general de la cuenta de almacenamiento seleccionada](./media/storage-insights-overview/storage-account-capacity-01.png)

## <a name="detailed-storage-workbooks"></a>Libros de almacenamiento detallados

Si seleccionó un valor en las columnas **Availability** (Disponibilidad), **E2E Latency** (Latencia de E2E), **Server Latency** (Latencia del servidor) y **transaction error type/Errors** (tipo de error de transacción/Errores) del libro **Overview** (Información general) de varias cuentas de almacenamiento, o seleccionó alguno de los botones de **Failures** (Errores), **Performance** (Rendimiento), **Availability** (Disponibilidad) y **Capacity** (Capacidad) en el libro **Overview** (Información general) de una cuenta de almacenamiento específica, cada uno de ellos proporciona un conjunto de información interactiva relacionada con el almacenamiento adaptada a esa categoría.  

* El botón **Disponibilidad** abre el libro **Disponibilidad**. En él se muestra el estado de mantenimiento actual del servicio Azure Storage, una tabla que muestra el estado de mantenimiento disponible de cada objeto clasificado por servicio de datos definido en la cuenta de almacenamiento con una línea de tendencia que representa el intervalo de tiempo seleccionado, y un gráfico de tendencias de disponibilidad para cada servicio de datos de la cuenta.  

    ![Ejemplo de informe de disponibilidad](./media/storage-insights-overview/storage-account-availability-01.png)

* **E2E Latency** (Latencia de E2E) y **Server Latency** (Latencia del servidor) abren el libro **Performance** (Rendimiento). Este libro incluye un icono de estado de acumulación que muestra la latencia de E2E y la latencia del servidor, un gráfico de rendimiento de E2E frente a la latencia del servidor y una tabla con el desglose de la latencia de las llamadas correctas por API clasificadas por servicio de datos definidas en la cuenta de almacenamiento.

    ![Ejemplo de informe de rendimiento](./media/storage-insights-overview/storage-account-performance-01.png)

* Al seleccionar cualquiera de las categorías de error enumeradas en la cuadrícula, se abre el libro **Failures** (Errores). El informe muestra los iconos de las métricas de todos los demás errores del cliente, excepto los descritos y las solicitudes correctas, los errores de limitación del cliente, un gráfico de rendimiento de la métrica de dimensión **Response Type** (Tipo de respuesta) de la transacción específica del atributo ClientOtherError y dos tablas, **Transactions by API name** (Transacciones por nombre de API) y **Transactions by Response type** (Transacciones por tipo de respuesta).

   ![Ejemplo de informe de errores](./media/storage-insights-overview/storage-account-failures-01.png)

* El botón **Capacity** (Capacidad) abre el libro **Capacity** (Capacidad). En él se muestran los iconos y el gráfico con la cantidad total de almacenamiento que se usa con cada objeto de datos de almacenamiento de la cuenta, y cuántos objetos de datos se almacenan en la cuenta.  

    ![Página de capacidad de la cuenta de almacenamiento seleccionada](./media/storage-insights-overview/storage-account-capacity-01.png) 

## <a name="pin-and-export"></a>Anclar y exportar

Cualquiera de las secciones de métricas se puede anclar un panel de Azure; para ello, seleccione el icono de chincheta en la parte superior derecha de la sección.

![Ejemplo de la sección de métricas anclada al panel](./media/storage-insights-overview/workbook-pin-example.png)

Los libros **Overview** (Información general) o **Capacity** (Capacidad) de varias suscripciones y cuentas de almacenamiento admiten la exportación de los resultados en formato Excel; para ello, seleccione el icono de flecha abajo situado a la derecha del icono de chincheta.

![Ejemplo de resultados de la cuadrícula de exportación de libros](./media/storage-insights-overview/workbook-export-example.png)

## <a name="customize-azure-monitor-for-storage"></a>Personalización de Azure Monitor para Storage

En esta sección se resaltan los escenarios comunes para editar el libro y personalizarlo de acuerdo con sus necesidades de análisis de datos:

* Definir el ámbito del libro para seleccionar siempre unas suscripciones o unas cuentas de almacenamiento determinadas
* Cambiar las métricas de la cuadrícula
* Cambiar el conjunto de disponibilidad
* Cambiar la representación de color

Las personalizaciones se guardan en un libro personalizado para evitar sobrescribir la configuración predeterminada del libro publicado. Los libros se guardan en un recurso compartido, bien en la sección **Mis informes** o en la sección **Informes compartidos** accesibles a cualquiera con acceso al grupo de recursos. Después de guardar el libro personalizado, debe ir a la galería de libros para iniciarlo.

![Inicio de la galería de libros desde la barra de comandos](./media/storage-insights-overview/workbook-command-bar-gallery.png)

### <a name="specifying-a-subscription-or-storage-account"></a>Especificación de una suscripción o una cuenta de almacenamiento

Puede configurar los libros **Overview** (Información general) o **Capacity** (Capacidad) de varias suscripciones y cuentas de almacenamiento para reducir el ámbito a una o varias suscripciones o cuentas de almacenamiento en concreto en cada ejecución. Para ello, realice los siguientes pasos.

1. Seleccione **Supervisión** en el portal y, luego, **Cuentas de almacenamiento** en el panel izquierdo.

2. En el libro **Overview** (Información general), elija **Edit** (Editar) en la barra de comandos.

3. En la lista **Subscriptions** (Suscripciones), seleccione una o varias suscripciones que quiere que sean las predeterminadas. Recuerde que el libro admite la selección de hasta 10 suscripciones.  

4. En la lista **Storage Accounts** (Cuentas de almacenamiento), seleccione una o varias cuentas que quiere que sean las predeterminadas. Recuerde que el libro admite la selección de hasta 200 cuentas de almacenamiento. 

5. Seleccione **Save as** (Guardar como) en la barra de comandos para guardar una copia del libro con las personalizaciones y, luego, haga clic en **Done editing** (Edición finalizada) para regresar el modo de lectura.  

### <a name="modify-metrics-and-colors-in-the-workbook"></a>Modificación de las métricas y los colores del libro

Los libros precompilados contienen datos de métricas y tiene la posibilidad de modificar o quitar cualquiera de las visualizaciones y personalizarlas según las necesidades específicas de su equipo humano.

En nuestro ejemplo, vamos a trabajar con el libro de capacidad de varias suscripciones y cuentas de almacenamiento con el objetivo de demostrar cómo:

* Quitar una métrica
* Cambiar la representación de color

Los mismos cambios se pueden realizar en cada uno de los libros precompilados de **Failures** (Errores), **Performance** (Rendimiento), **Availability** (Disponibilidad) y **Capacity** (Capacidad).

1. Seleccione **Supervisión** en el portal y, luego, **Cuentas de almacenamiento** en el panel izquierdo.

2. Seleccione **Capacity** (Capacidad) para cambiar al libro de capacidad y, en la barra de comandos, seleccione **Edit** (Editar).

    ![Selección de Edit (Editar) para modificar un libro](./media/storage-insights-overview/workbook-edit-workbook.png)

3. Junto a la sección de métricas, seleccione **Edit** (Editar).

    ![Selección de Edit (Editar) para modificar las métricas del libro de capacidad.](./media/storage-insights-overview/edit-metrics-capacity-workbook-01.png)

4. Vamos a quitar la columna **Account used capacity timeline** (Escala de tiempo de la capacidad usada por la cuenta), así que seleccione **Column Settings** (Configuración de columna) en la cuadrícula de métricas.

    ![Editar la configuración de las columnas](./media/storage-insights-overview/edit-capacity-workbook-resource-grid.png)

5. En el panel **Edit column settings** (Editar la configuración de las columnas), en la sección **Columns** (Columnas), seleccione **microsoft.storage/storageaccounts-Capacity-UsedCapacity Timeline$|Account used capacity Timeline$** (Escala de tiempo de la capacidad usada por la cuenta$), y, en la lista desplegable **Column renderer** (Representador de columnas), seleccione **Hidden** (Oculto).

6. Seleccione **Save and close** (Guardar y cerrar) para confirmar el cambio.

Ahora, vamos a cambiar el tema de color de las métricas de capacidad del informe para usar verde en lugar de azul.

1. Seleccione **Column Settings** (Configuración de columna) en la cuadrícula de métricas.

2. En el panel **Editar la configuración de las columnas**, en la sección **Columnas**, seleccione **microsoft.storage/storageaccounts-Capacity-UsedCapacity$`|`microsoft.storage/storageaccounts/blobservices-Capacity-BlobCapacity$`|`microsoft.storage/storageaccounts/fileservices-Capacity-FileCapacity$`|`microsoft.storage/storageaccounts/queueservices-Capacity-QueueCapacity$`|`microsoft.storage/storageaccounts/tableservices-Capacity-TableCapacity$** . En la lista desplegable **Color palette** (Paleta de colores), seleccione **Green** (Verde).

3. Seleccione **Save and close** (Guardar y cerrar) para confirmar el cambio.

4. Seleccione **Save as** (Guardar como) en la barra de comandos para guardar una copia del libro con las personalizaciones y, luego, haga clic en **Done editing** (Edición finalizada) para regresar el modo de lectura.  

### <a name="modify-the-availability-threshold"></a>Modificación del umbral de disponibilidad

En este ejemplo, vamos a trabajar con el libro de capacidad de la cuenta de almacenamiento y a demostrar cómo modificar el umbral de disponibilidad. De forma predeterminada, el icono y la cuadrícula que informan del porcentaje de disponibilidad se configuran con un umbral mínimo de 90 y un umbral máximo de 99. Vamos a cambiar el valor del umbral mínimo de la opción **Availability %** (% de disponibilidad) de la cuadrícula **Availability by API name** (Disponibilidad por nombre de API) al 85 %. Esto significa que el estado de mantenimiento cambia a crítico si el umbral es inferior al 85 por ciento. 

1. Seleccione **Cuentas de almacenamiento** en el portal y, luego, seleccione una cuenta de almacenamiento de la lista.

2. Seleccione **Información detallada** en el panel izquierdo.

3. En el libro, seleccione **Availability** (Disponibilidad) para cambiar al libro de disponibilidad y, luego, seleccione **Edit** (Editar) en la barra de comandos. 

4. Desplácese hacia abajo hasta la parte inferior de la página y, a la izquierda, junto a la cuadrícula **Availability by API** (Disponibilidad por API), seleccione **Edit** (Editar).

    ![Edición de la configuración de la cuadrícula Availability by API Name (Disponibilidad por nombre de API)](./media/storage-insights-overview/availability-workbook-avail-by-apiname.png)

5. Seleccione **Column settings** (Configuración de columna) y, en el panel **Edit column settings** (Editar la configuración de las columnas), en la sección **Columns** (Columnas) seleccione **Availability (%) (Thresholds + Formatted)** (Disponibilidad [%] [Umbrales y formato]).

6. Cambie el valor del estado de mantenimiento **Critical** (Crítico) de **90** a **85** y, luego, haga clic en **Save and Close** (Guardar y cerrar).

    ![Modificación del valor de umbral de disponibilidad de estado crítico](./media/storage-insights-overview/edit-column-settings-capacity-workbook-01.png)

7. Seleccione **Save as** (Guardar como) en la barra de comandos para guardar una copia del libro con las personalizaciones y, luego, haga clic en **Done editing** (Edición finalizada) para regresar el modo de lectura.

## <a name="troubleshooting"></a>Solución de problemas

Para obtener instrucciones generales para la solución de problemas, consulte el [artículo dedicado de solución de problemas](troubleshoot-workbooks.md) de conclusiones basadas en libros.

Esta sección le ayudará con el diagnóstico y la solución de algunos de los problemas habituales que pueden producirse en la aplicación al usar Azure Monitor para Storage. Utilice la siguiente lista para buscar la información relacionada con el problema específico.

### <a name="resolving-performance-capacity-or-availability-issues"></a>Resolución de problemas de rendimiento, capacidad o disponibilidad

Para ayudar a solucionar los problemas relacionados con el almacenamiento que identifique con Azure Monitor para Storage, consulte la [guía de solución de problemas](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#troubleshooting-guidance) de Azure Storage.  

### <a name="why-can-i-only-see-200-storage-accounts"></a>¿Por qué solo se pueden ver 200 cuentas de almacenamiento?

El número de cuentas de almacenamiento seleccionadas tiene un límite de 200, con independencia del número de suscripciones seleccionadas.

### <a name="how-to-change-the-coloring-and-threshold-for-availability"></a>¿Cómo se cambia el coloreado y el umbral para la disponibilidad?

Consulte la sección [Modificación del umbral de disponibilidad](#modify-the-availability-threshold) para ver los pasos detallados sobre cómo cambiar el coloreado y los umbrales para la disponibilidad.

### <a name="how-to-analyze-and-troubleshoot-the-data-shown-in-azure-monitor-for-storage"></a>¿Cómo se analizan y se solucionan problemas con los datos que se muestran en Azure Monitor para Storage?

 Consulte el artículo [Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md) para más información sobre cómo analizar y solucionar problemas de los datos de Azure Storage que se muestran en Azure Monitor para Storage.

### <a name="why-dont-i-see-all-the-types-of-errors-in-metrics"></a>¿Por qué no se ven todos los tipos de errores en las métricas?

Actualmente, se muestran hasta tres tipos diferentes de errores; el resto de los errores se reúnen en un único grupo. Esta opción se controla mediante splitByLimit y se puede modificar. Para cambiar esta propiedad:

1. Haga clic en Editar libro.
2. Vaya a Métricas, haga clic en Editar y, a continuación, seleccione **Transacciones, Suma** o cualquier métrica que desee editar.

    ![Vaya a Métricas y haga clic en Editar y, a continuación, en "Transacciones, Suma".](./media/storage-insights-overview/fqa7.png)

3. A continuación, cambie el valor en Número de divisiones.

    ![Seleccionar parámetros de métricas](./media/storage-insights-overview/fqa7-2.png)

Si desea ver n tipos diferentes de error, especifique splitByLimit como n + 1, 1 más para el resto de los errores.

###  <a name="i-saved-my-workbook-while-on-some-storage-account-why-cant-i-find-it-now"></a>Se ha guardado el libro con alguna cuenta de almacenamiento. ¿Por qué no se puede encontrar ahora?

Cada libro se guarda en la cuenta de almacenamiento desde la que se guardó. Intente buscar la cuenta de almacenamiento específica en la que el usuario guardó el libro. De lo contrario, no hay ninguna manera de encontrar un libro específico sin conocer el recurso (cuenta de almacenamiento).

## <a name="next-steps"></a>Pasos siguientes

* Configure [alertas de métricas](../platform/alerts-metric.md) y [notificaciones de estado del servicio](../../service-health/alerts-activity-log-service-notifications-portal.md) para generar alertas automáticas que ayuden a detectar los problemas.

* Conozca los escenarios para los que están concebidos los libros, cómo crear informes y personalizar los ya existentes y otros muchos temas en el artículo [Crear informes interactivos con libros de Azure Monitor](../platform/workbooks-overview.md).

* Para obtener orientación exhaustiva sobre el uso de análisis de almacenamiento y otras herramientas para identificar, diagnosticar y solucionar problemas relacionados con Azure Storage, consulte [Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md).

