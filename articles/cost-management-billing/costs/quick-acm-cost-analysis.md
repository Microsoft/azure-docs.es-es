---
title: 'Inicio rápido: Exploración de los costos de Azure con el análisis de costos'
description: Este guía de inicio rápido le ayudará a usar el análisis de costos para explorar y analizar sus costos de organización de Azure.
author: bandersmsft
ms.author: banders
ms.date: 08/20/2020
ms.topic: quickstart
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: micflan
ms.custom: seodec18
ms.openlocfilehash: 8ca9eaeeb8d484c9aeae6646e6f4814afbaed577
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718533"
---
# <a name="quickstart-explore-and-analyze-costs-with-cost-analysis"></a>Inicio rápido: Explore y analice los costos con Análisis de costos

Antes de que pueda controlar y optimizar correctamente sus costos de Azure, debe saber en qué lugar de su organización se originaron los costos. También resulta de utilidad saber cuánto dinero cuestan sus servicios y en apoyo a qué entornos y sistemas. La visión general de la gama de costos al completo es fundamental para conocer con precisión los patrones de gasto de la organización. Puede usar patrones de gastos para aplicar mecanismos de control de costos, como presupuestos.

En esta guía de inicio rápido, va a utilizar el análisis de costos para explorar y analizar los costos de organización. Puede ver los costos agregados de la organización para saber dónde se producen esos costos con el tiempo e identificar las tendencias de gasto. Puede ver los costos acumulados con el tiempo para estimar las tendencias de costos mensual, trimestral o incluso anualmente con respecto a un presupuesto. Un presupuesto ayuda a los proveedores a cumplir con las restricciones financieras. Y un presupuesto se utiliza para ver los costos diarios o mensuales para aislar las irregularidades de los gastos. Además, puede descargar los datos del informe actual para realizar análisis adicionales o para usarlos en un sistema externo.

En esta guía de inicio rápido, ha aprendido a hacer lo siguiente:

- Revise los costos en análisis de costos
- Personalización de vistas de costos
- Descargue datos de análisis de costos

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos es compatible con varios tipos de cuenta de Azure. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](understand-cost-mgt-data.md) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure.

Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../../cost-management/assign-access-acm-data.md).

Si su suscripción es nueva, no podrá usar inmediatamente las características de Cost Management. Para poder hacerlo deberán transcurrir un máximo de 48 horas.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

- Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="review-costs-in-cost-analysis"></a>Revise los costos en análisis de costos

Para revisar los costos en el análisis de costos, abra el ámbito en Azure Portal y seleccione **Análisis de costos** en el menú. Por ejemplo, vaya a **Suscripciones**, seleccione una suscripción de la lista y, a continuación, seleccione **Análisis de costos** en el menú. Use la píldora **Ámbito** para cambiar a un ámbito diferente en el análisis de costos. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

El ámbito que seleccione se usa en todo Cost Management para proporcionar la consolidación de datos y controlar el acceso a la información de costos. Cuando use ámbitos, no realice una selección múltiple de ellos. En su lugar, seleccione un ámbito mayor en el que otros se acumulan y, luego, filtre a los ámbitos anidados que necesite. Es importante comprender este enfoque, ya que algunas personas pueden no tener acceso a un solo ámbito principal que abarca múltiples ámbitos anidados.

Vea el vídeo [Uso de Cost Management en Azure Portal](https://www.youtube.com/watch?v=mfxysF-kTFA) para aprender a usar el análisis de costos. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/mfxysF-kTFA]

La vista de análisis de costos inicial incluye las siguientes áreas.

**Vista de costos acumulados**: representa la configuración de la vista del análisis de costos predefinida. Cada vista incluye la configuración del rango de fechas, granularidad, agrupar por y filtro. La vista predeterminada muestra los costos acumulados del período de facturación actual, pero se puede cambiar a otras vistas integradas. Para más información, consulte [Personalización de vistas de costos](#customize-cost-views).

**Costo real**: muestra los costos de compra y uso total del mes actual, a medida que se acumulan, y se muestran en la factura.

**Previsión**: muestra los costos totales previstos para el período que elija.

**Presupuesto**: muestra el límite de gasto previsto para el ámbito seleccionado, si está disponible.

**Granularidad acumulada**: muestra los costos diarios agregados totales desde el principio del período de facturación. Después de [crear un presupuesto](tutorial-acm-create-budgets.md) para su cuenta de facturación o suscripción, podrá ver rápidamente su tendencia de gastos con el presupuesto. Mantenga el puntero sobre una fecha para ver el costo acumulado hasta ese día.

**Gráficos dinámicos (anillos)** : proporcionan gráficos dinámicos, que desglosan el costo total por un conjunto común de propiedades estándar. Muestran los costos de mayor a menor del mes actual. Puede cambiar los gráficos dinámicos en cualquier momento seleccionando un área dinámica diferente. Los costos se clasifican por servicio (categoría de medidor), ubicación (región) y ámbito secundario predeterminado. Por ejemplo, las cuentas de inscripción se incluyen en las cuentas de facturación, los grupos de recursos se incluyen en las suscripciones y los recursos se incluyen en grupos de recursos.

![Vista inicial de análisis de costos en Azure Portal](./media/quick-acm-cost-analysis/cost-analysis-01.png)

### <a name="understand-forecast"></a>Descripción de la previsión

La previsión de costos muestra una proyección de los costos estimados durante el período de tiempo seleccionado. El modelo se basa en un modelo de regresión de series temporales. Se requiere al menos 10 días de datos de costo y uso recientes para predecir los costos de forma precisa. Para un período determinado, el modelo de previsión requiere la misma parte de datos de entrenamiento para el período de previsión. Por ejemplo, una proyección de tres meses necesita al menos tres meses de datos de costo y uso recientes.

El modelo utiliza un máximo de seis meses de datos de entrenamiento para proyectar los costos de un año. Como mínimo, necesita siete días de datos de entrenamiento para cambiar su predicción. La predicción se basa en cambios drásticos, como cambios abruptos e interrupciones, en patrones de costo y uso. La previsión no genera proyecciones individuales para cada elemento en las propiedades **Agrupar por** propiedades. Solo proporciona una previsión de los costos totales acumulados. Si usa varias monedas, el modelo proporciona una previsión de los costos solo en USD.

## <a name="customize-cost-views"></a>Personalización de vistas de costos

El análisis de costos tiene cuatro vistas integradas, optimizadas para los objetivos más comunes:

Ver | Responder preguntas como
--- | ---
Costo acumulado | ¿Cuánto he gastado hasta ahora en este mes? ¿Me mantendré dentro de mi presupuesto?
Costo diario | ¿Ha habido algún aumento en los costos por día durante los últimos 30 días?
Costo por servicio | ¿Cómo ha variado mi uso mensual en las últimas tres facturas?
Costo por recurso | ¿Qué recursos cuestan más en lo que va de mes?

![Selector de vista que muestra una selección de ejemplo de este mes](./media/quick-acm-cost-analysis/view-selector.png)

Sin embargo, hay muchos casos donde se necesitan análisis más profundos. La personalización comienza en la parte superior de la página, con la selección de fecha.

De forma predeterminada, el análisis de costos muestra los datos correspondientes al mes actual. Utilice el selector de fecha para cambiar rápidamente a los intervalos de fechas comunes. Algunos ejemplos son los últimos siete días, el mes pasado, el año actual o un intervalo de fechas personalizado. Las suscripciones de pago por uso también incluyen los intervalos de fechas según el período de facturación, que no está enlazado con el mes natural, como el período de facturación actual o la última factura. Use los vínculos  **<ANTERIOR** y  **SIGUIENTE>** en la parte superior del menú para ir al período anterior o siguiente, respectivamente. Por ejemplo, **<ANTERIOR** cambiará de **los últimos 7 días** a **8-14 días** o **15-21 días**.

![Selector de fecha que muestra una selección de ejemplo de este mes](./media/quick-acm-cost-analysis/date-selector.png)

De forma predeterminada, el análisis de costos muestra los costos **acumulados**. Los costos acumulados incluyen todos los costos correspondientes a cada día, además de los días anteriores, a fin de obtener una vista en constante crecimiento de sus costos agregados diarios. Esta vista está optimizada para mostrar sus tendencias con respecto a un presupuesto durante el intervalo de tiempo seleccionado.

Use la vista del gráfico para identificar posibles infracciones en el presupuesto. Cuando se produzca una posible infracción presupuestaria, el exceso de gasto proyectado se muestra en rojo. En el gráfico también se muestra un símbolo del indicador. Si se mueve el puntero sobre el símbolo, se muestra la fecha estimada de la infracción del presupuesto.

![Ejemplo que muestra una potencial infracción en el presupuesto](./media/quick-acm-cost-analysis/budget-breach.png)

También hay la vista **diaria** que muestra los costos de cada día. La vista diaria no muestra una tendencia de crecimiento. La vista está diseñada para mostrar irregularidades como picos de costos o caídas de un día para otro. Si ha seleccionado un presupuesto, la vista diaria también muestra una estimación del presupuesto diario.

Cuando los costos diarios son sistemáticamente superiores al presupuesto diario estimado, cabe esperar sobrepasar el presupuesto mensual. El presupuesto diario estimado es un medio para ayudarle a visualizar su presupuesto en un nivel más bajo. Cuando los costos diarios experimenten fluctuaciones, la comparación del presupuesto diario estimado con su presupuesto mensual será menos precisa.

Esta es una vista diaria de los gastos recientes con la previsión de gasto activada.
![Vista diaria que muestra costos diarios de ejemplo del mes actual](./media/quick-acm-cost-analysis/daily-view.png)

Si desactiva la previsión de gastos, no verá el gasto proyectado para fechas futuras. Además, al examinar los costos de los períodos anteriores, la previsión de costos no muestra los costos.

Por lo general, puede esperar ver datos o notificaciones para los recursos consumidos en un plazo de entre 8 y 12 horas.

**Agrupe por** propiedades comunes para desglosar los costos e identificar los principales colaboradores. Para agrupar por etiquetas de recursos, por ejemplo, seleccione la clave de etiqueta por la que quiera agrupar. Los costos se desglosan por cada valor de etiqueta, con un segmento adicional para los recursos que tengan dicha etiqueta aplicada.  Para obtener más información sobre las opciones de agrupación y filtrado, vea [Opciones de agrupación y filtrado](https://docs.microsoft.com/azure/cost-management-billing/costs/group-filter).

La mayoría de [recursos de Azure admiten etiquetado](../../azure-resource-manager/management/tag-support.md). Sin embargo, algunas etiquetas no están disponibles en Cost Management ni en facturación. Además, no se admiten las etiquetas de grupo de recursos. La compatibilidad con etiquetas se aplica al uso indicado *después* de que la etiqueta se aplicara al recurso. Las etiquetas no se aplican de forma retroactiva para los resúmenes de costos.

Consulte el vídeo sobre la [revisión de las directivas de etiquetas con Azure Cost Management](https://www.youtube.com/watch?v=nHQYcYGKuyw) para obtener información sobre el uso de una directiva de etiqueta de Azure para mejorar la visibilidad de los datos sobre costos.

Esta es una vista de los costos de los servicios de Azure del mes en curso.

![Vista acumulada diaria agrupada que muestra los costos de servicio de Azure de ejemplo correspondientes al último mes](./media/quick-acm-cost-analysis/grouped-daily-accum-view.png)

De manera predeterminada, el análisis de costos muestra todos los costos de compra y de uso cuando se acumulan y se mostrarán en su factura, también conocido como **Costo real**. Ver el costo real es ideal para la conciliación de la factura. Sin embargo, los picos de compras en el costo pueden ser alarmantes cuando se observan anomalías en los gastos y otros cambios en el costo. Para eliminar los picos causados por los costos de compra de reservas, cambie a **Costo amortizado**.

![Cambiar entre el costo real y el amortizado para ver las compras de reservas diseminadas por el plazo y asignadas a los recursos que usaron la reserva](./media/quick-acm-cost-analysis/metric-picker.png)

El costo amortizado desglosa las compras de reservas en fragmentos diarios y los distribuye a lo largo de la duración del plazo de la reserva. Por ejemplo, en lugar de ver una compra de 365 USD el 1 de enero, se verá una compra de 1,00 USD todos los días desde el 1 de enero al 31 de diciembre. Además de la amortización básica, estos costos también se reasignan y se asocian con los recursos específicos que usó la reserva. Por ejemplo, si ese cargo diario de 1,00 USD se divide entre dos máquinas virtuales, vería dos cargos de 0,50 USD cada día. Si una parte de la reserva no se usa en el día, se vería un cargo de 0,50 USD asociado con la máquina virtual aplicable y otro de la misma cantidad con el tipo de cargo de `UnusedReservation`. Tenga en cuenta que los costos de las reservas sin usar solo se pueden ver cuando se ve el costo amortizado.

Debido al cambio en la forma de representar los costos, es importante tener en cuenta que las vistas del costo real y del costo amortizado mostrarán números totales diferentes. En general, el costo total de los meses con una compra de reservas se reducirá cuando se ven los costos amortizados y los meses posteriores a una compra de reservas, aumentará. La amortización solo está disponible para las compras de reservas y, en este momento, no se aplica a las compras realizadas en Azure Marketplace.

La imagen anterior muestra los nombres de los grupos de recursos. Se puede realizar la agrupación por etiqueta para ver los costos totales por etiqueta, o bien usar la vista **Coste por recurso** para ver todas las etiquetas de un recurso determinado.

![Todos los datos de la vista actual donde se muestran los nombres de los grupos de recursos](./media/quick-acm-cost-analysis/full-data-set.png)

Al agrupar los costos por un atributo concreto, se muestran los diez mayores contribuidores a los costos, de mayor a menor. Si hay más de diez grupos, se muestran los nueve mayores contribuidores a los costos, así como un grupo **Otros**, que representa todos los grupos restantes combinados. Al agrupar por etiquetas, se muestra un grupo **Sin etiqueta** para los costos a los que no se ha aplicado la clave de etiqueta. El grupo **Sin etiqueta** siempre es el último, aunque haya más costos sin etiqueta que con ella. Si hay 10 o más valores de etiqueta, los costos sin etiqueta formarán parte de **Otros**. Cambie a la vista de tabla y cambie la granularidad a **Ninguno** para ver todos los valores clasificados de mayor a menor costo.

Las máquinas virtuales, redes y recursos de almacenamiento clásicos no comparten datos de facturación detallados. Al agrupar los costos, se combinan como **servicios clásicos**.

Los gráficos dinámicos del gráfico principal muestran distintas agrupaciones que le ofrecen una imagen más amplia de los costos generales durante el período de tiempo seleccionado y con los filtros elegidos. Seleccione una propiedad o una etiqueta para ver los costos agregados por cualquier dimensión.

![Ejemplo que muestra gráficos dinámicos](./media/quick-acm-cost-analysis/pivot-charts.png)

Puede ver el conjunto de datos completo de cualquier vista. Todas las selecciones o los filtros que aplique afectan a los datos presentados. Para ver el conjunto de datos completo, haga clic en la lista **Tipo de gráfico** y, después, haga clic en la vista **Tabla**.

![Datos de la vista actual en una vista de tabla](./media/quick-acm-cost-analysis/chart-type-table-view.png)

## <a name="saving-and-sharing-customized-views"></a>Almacenamiento y uso compartido de vistas personalizadas

Para guardar y compartir vistas personalizadas con otros usuarios, ancle el análisis de costos al panel de Azure Portal o copie un vínculo al análisis de costos.

Vea el vídeo [Compartir y guardar vistas en Azure Cost Management](https://www.youtube.com/watch?v=kQkXXj-SmvQ) para aprender a usar el portal para compartir la información de los costos en la organización. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/kQkXXj-SmvQ]

Para anclar un análisis de costos, seleccione el icono de anclaje en la esquina superior derecha. El anclaje de un análisis de costos solo guardará el gráfico principal o la vista de tabla. Comparta el panel para dar a otros usuarios acceso al icono. Tenga en cuenta que así solo se comparte la configuración del panel y no se concede a otros usuarios acceso a los datos subyacentes. Si no tiene acceso a los costos, pero tiene acceso a un panel compartido, verá un mensaje de acceso denegado.

Para compartir un vínculo al análisis de costos, seleccione **Compartir** en la parte superior de la hoja. Se mostrará una dirección URL personalizada, que abre esta vista específica para este ámbito concreto. Si no tiene acceso de costo y obtiene esta dirección URL, verá el mensaje "acceso denegado".

Para más información acerca de cómo conceder acceso a los costos para cada ámbito compatible, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

## <a name="download-usage-data"></a>Descarga de datos de uso

Hay ocasiones en que es necesario descargar los datos para analizarlos en mayor profundidad, combinarlos con los datos propios o integrarlos en los sistemas propios. Cost Management ofrece algunas opciones distintas. Como punto de partida, si necesita un resumen ad hoc de alto nivel, como el que se obtiene en el análisis de costos, cree la vista que necesite. Después, descárguelo seleccionando **Exportar** y **Descargar datos en CSV** o **Descargar datos en Excel**. La descarga de Excel proporciona contexto adicional en la vista que usó para generar la descarga, como el ámbito, la configuración de las consultas, el total y la fecha en que se generó.

Si necesita el conjunto de datos completo sin agregar, descárguelo de la cuenta de facturación. A continuación, en la lista de servicios del panel de navegación izquierdo del portal, vaya a **Administración de costos + facturación**. Seleccione su cuenta de facturación, si procede. Vaya a **Uso y cargos** y, a continuación, seleccione el icono de **descarga** para el período de facturación deseado.

## <a name="clean-up-resources"></a>Limpieza de recursos

- Si ha anclado una vista personalizada para el análisis de costos, pero ya no la necesita, vaya al panel en el que la ancló y elimine la vista anclada.
- Si ha descargado los archivos de datos de uso y ya no los necesita, asegúrese de eliminarlos.

## <a name="next-steps"></a>Pasos siguientes

Continúe con el primer tutorial para aprender crear y administrar presupuestos.

> [!div class="nextstepaction"]
> [Creación y administración de presupuestos](tutorial-acm-create-budgets.md)
