---
title: Usos del análisis de costos comunes en Azure Cost Management
description: En este artículo se explica cómo puede obtener los resultados de las tareas comunes de análisis de costos en Azure Cost Management.
author: bandersmsft
ms.author: banders
ms.date: 07/24/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.openlocfilehash: c03bf5e52ae7f6b259c7b744e6033d760af23dd0
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683681"
---
# <a name="common-cost-analysis-uses"></a>Usos del análisis de costos comunes

Los usuarios de Azure Cost Management suelen querer respuesta a preguntas que muchos otros formulan. En este artículo se explica cómo obtener los resultados de las tareas comunes de análisis de costos en Cost Management.

## <a name="view-forecasted-costs"></a>Ver costos previstos

Los costos previstos se muestran en las áreas de análisis de costos para las vistas de área y columna apilada. La previsión se basa en el uso de recursos históricos. Los cambios en el uso de recursos afectan a los costos previstos.

En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.

En la vista predeterminada, el gráfico superior tiene las secciones de costo real o amortizado y costo previsto. El color sólido del gráfico muestra el costo real o amortizado. El color sombreado muestra el costo de la previsión.

[![Costos previstos](./media/cost-analysis-common-uses/enrollment-forecast.png)](./media/cost-analysis-common-uses/enrollment-forecast.png#lightbox)

## <a name="view-forecasted-costs-grouped-by-service"></a>Visualización de los costos previstos agrupados por servicio

La vista predeterminada no muestra el grupo de costos previstos por un servicio, por lo que tiene que agregar un grupo por selección.

En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.

Seleccione **Agrupar por** > **Nombre de servicio**.

La vista muestra los costos agrupados de cada servicio. El costo previsto no se calcula para cada servicio. Está proyectado para el **total** de todos los servicios.

[![Costo previsto agrupado](./media/cost-analysis-common-uses/forecast-group-by-service.png)](./media/cost-analysis-common-uses/forecast-group-by-service.png#lightbox)

## <a name="view-forecasted-costs-for-a-service"></a>Visualización de los costos previstos de un servicio

Puede ver los costos previstos delimitados a un único servicio. Por ejemplo, puede que desee ver los costos previstos solo para máquinas virtuales.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Seleccione **Agregar filtro** y, después, seleccione **Nombre de servicio**.
1. En la lista **Elegir**, seleccione un servicio. Por ejemplo, seleccione **Máquinas virtuales**.

Revise el costo real de la selección y el costo previsto.

Puede agregar más personalizaciones a la vista.

1. Agregue un segundo filtro para **Medidor** y seleccione un valor para filtrar por un tipo de medidor individual en el nombre de servicio seleccionado.
1. Agrupe por **Recurso** para ver los recursos específicos que se acumulan en el costo. El costo previsto no se calcula para cada servicio. Está proyectado para el **total** de todos los recursos.

[![Costo previsto de un servicio](./media/cost-analysis-common-uses/forecast-by-service.png)](./media/cost-analysis-common-uses/forecast-by-service.png#lightbox)

## <a name="view-your-azure-and-aws-costs-together"></a>Visualización de los costos de Azure y AWS juntos  

Para ver los costos de Azure y AWS juntos, se usan ámbitos del grupo de administración en Azure.

1. Cree un grupo de administración o seleccione uno existente.
1. Asigne las suscripciones de Azure existentes que necesita al grupo de administración.
1. Asigne el *mismo* grupo de administración a la cuenta vinculada del conector.
1. Vaya a Análisis de costos y seleccione **Costos acumulados**.
1. Seleccione **Agrupar por** - **Proveedor**.

## <a name="view-cost-breakdown-by-azure-service"></a>Visualización del desglose de costos por servicio de Azure

La visualización de los costos por un servicio de Azure puede ayudarle a conocer mejor qué partes de su infraestructura son las más caras. Por ejemplo, los costos de proceso de la máquina virtual pueden ser pequeños. Aún así, es posible que se acumulen costos de red significativos debido a la cantidad de información que las máquinas virtuales emiten. Conocer los principales costos de los servicios de Azure es esencial para ajustar el uso del servicio según sea necesario.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Seleccione **Costo por servicio** y, después, agrupe por **Nivel de servicio**.
1. Cambie la vista a **Tabla**.

[![Visualización del desglose de costos por servicio de Azure](./media/cost-analysis-common-uses/breakdown-by-service.png)](./media/cost-analysis-common-uses/breakdown-by-service.png#lightbox)

## <a name="review-invoiced-charges-in-cost-analysis"></a>Revisión de los cargos facturados en el Análisis de costos

Para ver los detalles de la factura en Azure Portal, vaya a Análisis de costos para el ámbito asociado a la factura que está analizando. Cambie a la vista **Detalles de la factura**. Los detalles de la factura muestran los cargos según se muestran en la factura.

[![Ejemplo que muestra los detalles de la factura](./media/cost-analysis-common-uses/invoice-details.png)](./media/cost-analysis-common-uses/invoice-details.png#lightbox)

Al ver los detalles de la factura, puede identificar el servicio que tiene costos inesperados y determinar qué recursos se asocian directamente con el recurso en Análisis de costos. Por ejemplo, si desea analizar los cargos del servicio Virtual Machines, vaya a la vista **Costo acumulado**. A continuación, establezca la granularidad en **Diario** y filtre los cargos **Nombre de servicio: Máquinas virtuales** y agrupe los cargos por **Recurso**.

[![Ejemplo que muestra los costos acumulados de las máquinas virtuales](./media/cost-analysis-common-uses/virtual-machines.png)](./media/cost-analysis-common-uses/virtual-machines.png#lightbox)

## <a name="view-cost-breakdown-by-azure-resource"></a>Visualización del desglose de costos por recurso de Azure

Los servicios se compilan con los recursos de Azure. Revisar los costos en función de los recursos puede ayudarle a identificar rápidamente quiénes contribuyen al costo principal. Si un servicio tiene recursos que son demasiado caros, considere la posibilidad de realizar cambios para reducir los costos.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Seleccione **Costo por recurso**.
1. Cambie la vista a **Tabla**.

[![Visualización del desglose de costos por recurso de Azure](./media/cost-analysis-common-uses/cost-by-resource.png)](./media/cost-analysis-common-uses/cost-by-resource.png#lightbox)

## <a name="view-cost-breakdown-by-selected-dimensions"></a>Visualización del desglose de costos por dimensiones seleccionadas

Las dimensiones permiten organizar los costos en función de los distintos valores de metadatos que se muestran en los cargos. Por ejemplo, podría agrupar los costos por ubicación.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Seleccione el filtro **Agrupar por**.  
    [![Selección de un elemento Agrupar por](./media/cost-analysis-common-uses/group-by.png)](./media/cost-analysis-common-uses/group-by.png#lightbox)
1. También puede guardar la vista para usarla después.
1. Haga clic en un gráfico circular debajo del gráfico para ver datos más detallados.  
    [![Visualización del desglose de costos por dimensiones seleccionadas](./media/cost-analysis-common-uses/drill-down.png)](./media/cost-analysis-common-uses/drill-down.png#lightbox)

## <a name="view-costs-per-day-or-by-month"></a>Ver costos por día o por mes

Examinar los costos diarios y mensuales puede ayudarle a comprender mejor si hay una hora de la semana o del año en la que los costos son mayores. Si tiene más tráfico de clientes en un período de vacaciones, ¿eso conlleva un aumento correspondiente en los costos de Azure? ¿Es el viernes un día más caro que el lunes?

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Establezca la **Granularidad** en **Mensual** o **Diaria**.

[![Ver costos por día](./media/cost-analysis-common-uses/daily-granularity.png)](./media/cost-analysis-common-uses/daily-granularity.png#lightbox)


## <a name="view-your-spot-vm-charges"></a>Visualización de los cargos de máquinas virtuales de Spot

Las máquinas virtuales de Spot pueden proporcionar un gran ahorro en los costos de cargas de trabajo que pueden controlar las interrupciones. Las cargas de trabajo se ejecutan sobre capacidad de Azure sin usar. Dado que se pueden desalojar en cualquier momento, las máquinas virtuales de Spot proporcionan un descuento significativo. Use los siguientes pasos para ver los cargos de las máquinas virtuales de Spot.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo, **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
2. Agregue un filtro para **Modelo de precios: Spot**.

![Ejemplo que muestra el filtro de máquinas virtuales de Spot](./media/cost-analysis-common-uses/spot-vm-filter.png)

La dimensión del modelo de precios también se usa para ver cargos de reserva y a petición.

## <a name="view-your-reservation-charges"></a>Visualización de los cargos de reserva

Las instancias reservadas proporcionan una manera de ahorrar dinero con Azure. Con las reservas, gastará dinero por adelantado para un número determinado de recursos a lo largo del tiempo. El análisis de costos muestra los cargos según aparecen en la factura. Los cargos se muestran como costos reales u amortizados en el transcurso del período de reserva.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo, **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Agregue un filtro para **Modelo de precios: Reserva**.
1. En **Ámbito**, junto al costo que se muestra, haga clic en el símbolo de flecha hacia abajo y seleccione la métrica **Costo real** o **Costo amortizado**.

![Selección de una métrica de costo](./media/cost-analysis-common-uses/metric-cost.png)

Cada métrica afecta al modo en que se muestran los datos de los cargos de reserva.

**Costo real**: muestra la compra tal como aparece en la factura. Por ejemplo, si compró una reserva de un año por 1200 USD en enero, el análisis de costos muestra un costo de 1200 USD en el mes de enero para la reserva. No muestra un costo de reserva para otros meses del año. Si agrupa los costos reales por máquina virtual, una máquina virtual que haya recibido la ventaja de la reserva para un mes determinado tendrá un costo de cero para ese mes.

**Costo amortizado**: muestra la compra de reserva dividida como un costo amortizado a lo largo del período de reserva. Con el mismo ejemplo anterior, si adquirió una reserva de un año por 1200 USD en enero, el análisis de costos muestra un costo de 100 USD para cada mes a lo largo del año. Si agrupara los costos por máquina virtual en este ejemplo, vería el costo atribuido a cada máquina virtual que haya recibido la ventaja de la reserva.

## <a name="view-your-reservation-utilization"></a>Visualización de la utilización de la reserva

Después de comprar una reserva, es importante realizar un seguimiento de su uso para saber que recibe lo que ha pagado. Por ejemplo, si compró 10 máquinas virtuales durante un año y solo usa cinco de ellas, básicamente la mitad de la compra se desperdicia. Hay dos maneras diferentes de evaluar la utilización:

### <a name="view-unused-ri-costs-in-cost-analysis"></a>Visualización de costos de instancia reservada sin usar en el análisis de costos

Siga los pasos que se indican a continuación para identificar cuántos costos se desperdician actualmente cada mes en la compra de reserva.

1. En Azure Portal, vaya al análisis de costos para el ámbito en el que se aplica su reserva. Por ejemplo, **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Agregue un filtro para **Modelo de precios: Reserva**.
1. Seleccione la vista **Costo amortizado**.
1. Establezca la granularidad en **Mensual**.
1. Establezca el período de tiempo en el año actual o en el período de la reserva.
1. Establezca el tipo de gráfico en **Columna (apilada)** .
1. Agrupe los cargos por **Tipo de cargo**.
1. Revise los resultados de los valores de `unusedreservation`.

[![Ejemplo que muestra el uso de la reserva](./media/cost-analysis-common-uses/view-reservation-cost.png)](./media/cost-analysis-common-uses/view-reservation-cost.png#lightbox)

### <a name="view-utilization-in-reservations"></a>Visualización del uso de las reservas

Para obtener instrucciones detalladas, consulte [Optimización del uso de reservas](../reservations/manage-reserved-vm-instance.md#optimize-reservation-use).

## <a name="view-costs-for-a-specific-tag"></a>Ver los costos de una etiqueta específica

Muchos usuarios de Azure aplican etiquetas a sus recursos, como un centro de costos o un entorno de desarrollo (producción y pruebas), para clasificar mejor los cargos. Las etiquetas aparecen como una dimensión en el análisis de costos. Puede usar la dimensión para obtener información detallada sobre las categorías de etiquetas personalizadas.

La compatibilidad con etiquetas se aplica al uso indicado *después* de que la etiqueta se aplicara al recurso. Las etiquetas no se aplican de forma retroactiva para los resúmenes de costos.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Seleccione **Agrupar por** para la etiqueta.

[![Visualización de los costos de una etiqueta específica](./media/cost-analysis-common-uses/tag.png)](./media/cost-analysis-common-uses/tag.png#lightbox)

## <a name="download-your-usage-details"></a>Descarga de los detalles de uso

El archivo de informe de detalles de uso, en formato CSV, proporciona un desglose de todos los cargos que se acumulan en una factura. Puede usar el informe para compararlo con la factura y comprenderla mejor. Cada cargo de la factura corresponde a los cargos desglosados en el informe de uso.

1. En Azure Portal, vaya a la pestaña **Usage and Charges** (Uso y cargos) para una cuenta de facturación o una suscripción. Por ejemplo: **Administración de costos + facturación** > **Facturación** > **Usage + charges** (Uso y cargos).
1. Seleccione el elemento de la línea del que desea descargar y, a continuación, haga clic en el símbolo de descarga.  
    [![Descarga del uso y los cargos](./media/cost-analysis-common-uses/download1.png)](./media/cost-analysis-common-uses/download1.png#lightbox)
1.  Seleccione el archivo de uso que se va a descargar.  
    ![Elección del archivo de uso que se va a descargar](./media/cost-analysis-common-uses/download2.png)

## <a name="view-monthly-ea-cost-breakdown"></a>Visualización del desglose mensual del costo de EA

La inscripción de EA acumula los costos de toda la organización. Entender cómo se acumulan los costos y se facturan a lo largo del tiempo ayuda a atraer a las partes interesadas adecuadas para garantizar que se administren de forma responsable.

Solo se muestran los costos de la inscripción activa. Si transfirió una inscripción (inactiva) a una nueva (activa), los costos de la inscripción anterior no se muestran en Cost Management.


1. En Azure Portal, vaya a **Administración de costos + facturación** > **Información general**.
1. Haga clic en **Desglose** para el mes en curso y vea la evolución del compromiso monetario.  
    [![Información general sobre costos de EA: desglose](./media/cost-analysis-common-uses/breakdown1.png)](./media/cost-analysis-common-uses/breakdown1.png#lightbox)
1.  Haga clic en la pestaña **Usage and Charges** (Uso y cargos) y vea el desglose del mes anterior en el intervalo de tiempo seleccionado.  
    [![Pestaña Usage and charges (Uso y cargos)](./media/cost-analysis-common-uses/breakdown2.png)](./media/cost-analysis-common-uses/breakdown2.png#lightbox)

## <a name="view-enrollment-monthly-cost-by-term"></a>Visualización del costo mensual de inscripción por trimestre

Use una vista gráfica de los costos mensuales de su inscripción para conocer las tendencias de costos y los importes facturados durante un período determinado.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Seleccione su inscripción y establezca su trimestre.
1. Establezca una granularidad mensual y, a continuación, establezca la vista en **Columna (apilada)** .

Puede agrupar y filtrar los datos para un análisis más detallado.

[![Costo de inscripción mensual por trimestre](./media/cost-analysis-common-uses/enrollment-term1.png)](./media/cost-analysis-common-uses/enrollment-term1.png#lightbox)

## <a name="view-ea-enrollment-accumulated-costs"></a>Visualización de los costos acumulados de inscripción de EA

Vea los cargos acumulados netos a lo largo del tiempo para conocer los gastos generales de su organización durante un período determinado.

1. En Azure Portal, vaya al análisis de los costos para su ámbito. Por ejemplo: **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**.
1. Seleccione su inscripción y, a continuación, vea los costos acumulados actuales.

[![Costos acumulados de inscripción](./media/cost-analysis-common-uses/cost-analysis-enrollment.png)](./media/cost-analysis-common-uses/cost-analysis-enrollment.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes
- Si aún no ha completado la primera guía rápida de Cost Management, léala en [Start analyzing costs](quick-acm-cost-analysis.md) (Comenzar a analizar los costos).
- Lea la [documentación sobre Cost Management](../index.yml).
