---
title: Uso de informes de Cloudyn en Azure
description: En este artículo se describe el propósito de los informes de Cloudyn que se incluyen en el portal de Cloudyn para ayudarlo a usarlas de manera eficaz.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 4bc68e851c4f3ba85f8ea18a91d86906e1f87887
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88685228"
---
# <a name="reports-available-in-the-cloudyn-portal"></a>Informes disponibles en el portal de Cloudyn

En este artículo se describe el propósito de los informes de Cloudyn que se incluyen en el portal de Cloudyn. También se describe cómo puede usar eficazmente los informes. La mayoría de los informes son intuitivos y tienen un aspecto uniforme. La mayor parte de las acciones que puede realizar en un informe también puede hacerlas en otros informes. Para información general sobre cómo usar los informes de Cloudyn, lo que incluye cómo personalizarlos, guardarlos o programarlos, consulte [Informes de costos](understanding-cost-reports.md).

Azure Cost Management ofrece una funcionalidad similar a Cloudyn. Azure Cost Management es una solución de administración de costos de Azure nativa. Le ayuda a analizar costos, crear y administrar presupuestos, exportar datos y revisar y actuar sobre recomendaciones de optimización para ahorrar dinero. Para más información, consulte [Azure Cost Management](../cost-management-billing-overview.md).

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="report-types"></a>Tipos de informes

Existen tres tipos de informes de Cloudyn:

- Informes a lo largo del tiempo. Por ejemplo, el informe de costo a lo largo del tiempo. Los informes a lo largo del tiempo muestran una serie temporal de datos a lo largo de un intervalo seleccionado con una resolución predefinida, y muestran una resolución semanal correspondiente a los dos últimos meses. Puede usar agrupación y filtrado para acercar varios puntos de datos.
  - Los informes a lo largo del tiempo pueden ayudarle a ver las tendencias y a detectar picos o anomalías.
- Informes de análisis. Por ejemplo, el informe de análisis de costos. Estos informes muestran los datos agregados durante un período definido y permiten la agrupación y el filtrado de los datos.
  - Los informes de análisis pueden ayudarle a ver los picos y a determinar las causas de anomalías; también muestran un desglose pormenorizado de los datos.
- Informes tabulares. Puede ver cualquier informe como una tabla, pero algunos informes se ven solo como una tabla. Estos informes proporcionan listas detalladas de elementos.
  - Las recomendaciones son informes tabulares, no hay visualizaciones para las recomendaciones. Sin embargo, puede visualizar los resultados de la recomendación. Por ejemplo, los ahorros con el transcurso del tiempo.
  - Los informes tabulares son útiles como listas de acciones o para exportar datos a fin de procesarlos posteriormente. Por ejemplo, un informe de contracargo.

Los informes de costos muestran costos _reales_ o _amortizados_.

Los informes de costo real muestran los pagos realizados durante el período de tiempo seleccionado. Por ejemplo, todos los cargos únicos, como compras de instancias reservadas (RI) se muestran en los informes de costo real como picos de costo.

Los informes de costo amortizado distribuyen los cargos únicos durante un período en el que se aplican. Por ejemplo, los cargos únicos por las compras de instancias reservadas se distribuyen durante el plazo de reserva y no se muestran como un pico. La vista amortizada es la única forma de ver las tendencias reales y realizar previsiones de costos.

En algunos casos, la amortización se presenta como un informe independiente. Por ejemplo, los informes de análisis de costo y análisis de costo amortizado. En otros casos, la amortización es una directiva de informes, como los informes de asignación de costos y análisis de costos.

Puede programar cualquier informe para la entrega periódica. Los informes de costos permiten establecer un umbral, por lo que son útiles para las alertas.

## <a name="cost-analysis-vs-cost-allocation"></a>Análisis de costos frente a asignación de costos

En los informes de _análisis de costos_ se muestran datos de facturación de los proveedores de nube. Con los informes, puede agrupar y profundizar en diversos segmentos de datos detallados del archivo de facturación. Los informes permiten desplazarse por los costos pormenorizados en los datos de facturación sin procesar de los proveedores de nube.

Algunos informes de _análisis de costos_ no agrupan los costos por etiquetas de recursos. Y, la información de facturación basada en etiquetas solo aparece en los informes después de asignar los costos mediante la creación de un modelo de costos con la [asignación de costos 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs).

Los informes de _asignación de costos_ están disponibles tras crear un modelo de costos mediante la [asignación de costos 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs). Cloudyn procesa datos de costos o facturación y hace _coincidir_ los datos con los datos de uso y etiqueta datos de sus cuentas en la nube. Para hacer coincidir los datos, Cloudyn requiere acceso a sus datos de uso. Si tiene cuentas que carecen de credenciales, se etiquetan como _recursos no clasificados_.

## <a name="dashboards"></a>Paneles

Los paneles de Cloudyn proporcionan una visión general de los informes. Los paneles se componen de widgets, y cada widget es básicamente una miniatura del informe. Al [personalizar los informes](understanding-cost-reports.md#save-and-schedule-reports), se guardan en Mis informes y se agregan al panel. Para más información sobre los paneles, consulte [Vista de las métricas clave de costos con paneles](dashboards.md).

## <a name="budget-information-in-reports"></a>Información del presupuesto en informes

Muchos informes de Cloudyn muestran información de presupuesto después de que se ha creado manualmente uno. Así que, hasta que no cree un presupuesto, los informes no mostrarán información al respecto. Para más información, consulte el artículo sobre la [configuración de la administración de presupuesto](#budget-management-settings).

## <a name="reports-and-reporting-features"></a>Informes y características de informes

Cloudyn incluye los siguientes informes y características de informes.

### <a name="cost-navigator-report"></a>Informe de Navegador de costos

El informe de Navegador de costos es una forma rápida de ver el consumo de facturación mediante una vista de panel. Presenta un subconjunto de filtros y vistas básicas para mostrar inmediatamente una vista resumida de los costos de la organización. Los costos se muestran por fecha. Dado que el informe pretende ser una vista inicial de los costos, no es tan flexible y completo como muchos otros informes o paneles personalizados que puede crear por su cuenta.

De forma predeterminada, las vistas principales del informe muestran:

- El costo a lo largo de tiempo con una vista de gráfico de barras de semana laboral. Puede cambiar el **intervalo de fechas** para cambiar el gráfico de barras del intervalo de fechas.
- Gastos por servicio, mediante un gráfico circular.
- Clasificación de recursos por etiquetas, mediante un gráfico circular.
- Gastos por entidades de costo, mediante un gráfico circular.
- Costo total, por fecha en una vista de lista.

### <a name="cost-analysis-report"></a>Informe de análisis de costos

El informe de análisis de costos es un cálculo de visualización y contracargo, basado en la directiva. Agrega el consumo en la nube durante un período de tiempo seleccionado, tras haber aplicado al costo todas las reglas de asignación. Por ejemplo, calcula los costos por etiquetas, reasigna los costos de recursos sin etiquetar y, opcionalmente, asigna el uso de instancias reservadas.

Las directivas establecidas en la [asignación de costos 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs) se usan en el informe de análisis de costos y los resultados se combinan entonces con información de los datos sin procesar del proveedor de nube.

¿Cómo se calcula este informe? El servicio Cloudyn garantiza que la asignación retenga la integridad de cada cuenta vinculada mediante la aplicación de la _afinidad de la cuenta_. La afinidad garantiza que una cuenta que no usa un servicio específico no tenga asignado ningún costo de este servicio. Los costos acumulados en esa cuenta permanecen allí y las directivas de asignación no los calculan. Por ejemplo, podría tener cinco cuentas vinculadas. Si solo tres de ellas usan servicios de almacenamiento, el costo de los servicios de almacenamiento solo se asigna entre etiquetas en las tres cuentas.

Use el informe de análisis de costos para:

- Calcular el contracargo/visualización de su organización
- Clasificar todos los costos
- Mostrar una vista agregada de toda su implementación durante un período de tiempo específico.
- Consultar los costos por categorías de etiquetas en función de las directivas creadas en el modelo de costos.

Para usar el informe de análisis de costos:

1. Seleccione un intervalo de fechas.
2. Agregue etiquetas, según sea necesario.
3. Agregue grupos.
4. Elija un modelo de costos creado anteriormente.

### <a name="cost-over-time-report"></a>Informe de costo a lo largo del tiempo

El informe de costo a lo largo del tiempo muestra los resultados de la asignación de costos como una serie temporal. Permite observar tendencias y detectar irregularidades en su implementación. En el se muestran básicamente los costos distribuidos a lo largo de un período definido. En el informe se muestran sus colaboradores de costos principales, incluidos los costos en curso y los cargos únicos de las instancias reservadas que se gastan durante un período de tiempo seleccionado. En este informe se usan las directivas establecidas en la [asignación de costos 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs).

Use el informe de costo a lo largo del tiempo para:

- Consultar los cambios a lo largo del tiempo y qué influencias cambian de un día (o intervalo de fechas) para otro.
- Analizar los costos a lo largo del tiempo para una instancia específica.
- Comprender por qué hubo un aumento del costo para una instancia específica.

Para usar el informe de costo a lo largo del tiempo:

1. Seleccione un intervalo de fechas.
2. Agregue etiquetas, según sea necesario.
3. Agregue grupos.
4. Elija un modelo de costos creado anteriormente.
5. Seleccione los costos reales o amortizados.
6. Elija si desea aplicar reglas de asignación para consultar la vista de datos de facturación sin formato o la vista para recalcular el costo.

### <a name="actual-cost-analysis-report"></a>Informe de análisis de costo real

El informe de análisis de costo real muestra los costos del proveedor sin modificaciones. En él aparecen los principales contribuyentes a los costos, incluidos los costos en curso y los cargos únicos.

Puede usar el informe para ver la información de costo de las suscripciones. En el informe, las suscripciones de Azure se muestran como **nombre de cuenta** y **número de cuenta**. Las **cuentas vinculadas** muestran suscripciones de AWS. Para ver los costos por suscripción, un desglose para cada cuenta, en **Grupos**, seleccione el tipo de suscripción que tenga.

Use el informe de análisis del costo real para:

- Analizar y supervisar los costos del proveedor sin procesar gastados durante un período de tiempo específico
- Programar una alerta de umbral
- Analizar los costos sin modificaciones producidos por las cuentas y entidades.

### <a name="actual-cost-over-time-report"></a>Informe de costo real a lo largo del tiempo

El informe de costo real a lo largo del tiempo es un informe de análisis del costo estándar que distribuye el costo en una resolución temporal definida. En el informe se muestra el gasto a lo largo del tiempo, lo que le permite observar tendencias y detectar irregularidades relativas al gasto. En este informe se muestran sus colaboradores de costos principales, incluidos los costos en curso y los cargos únicos de las instancias reservadas que se gastan durante un período de tiempo seleccionado.

Use el informe de costo real a lo largo del tiempo para:

- Ver las tendencias de costos a lo largo del tiempo.
- Buscar irregularidades en el costo.
- Encontrar todas las preguntas relativas a los costos que tienen que ver con los proveedores de nube.

### <a name="amortized-cost-reports"></a>Informes de costo amortizado

En este conjunto de informes de costo amortizado se muestran los cargos por servicios no basados en el uso linealizados o los costos pagaderos una sola vez y se reparte su costo a lo largo del tiempo de manera uniforme durante su vida útil. Por ejemplo, los cargos únicos podrían incluir:

- Cargos anuales por soporte técnico
- Cargos anuales por componentes de seguridad
- Cargos de compra de instancias reservadas
- Algunos elementos de Azure Marketplace

En el archivo de facturación, los cargos únicos se aplican en los casos en que las fechas de inicio y finalización (marca de tiempo) del consumo de servicios tienen los mismos valores. El servicio Cloudyn los reconoce entonces como cargos únicos que se amortizan. Otros servicios basados en el consumo con costos de uso a petición no se amortizan.

Los informes de costo amortizado incluyen:

- Análisis de costo amortizado
- Costo amortizado a lo largo del tiempo

### <a name="cost-analysis-report"></a>Informe de análisis de costos

En el informe de análisis de costos se proporciona información sobre el consumo y el gasto en la nube durante un período de tiempo seleccionado. Las directivas establecidas en la [asignación de costos 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs) se usan en el informe de análisis de costos.

¿Cómo calcula Cloudyn este informe?

Cloudyn garantiza que la asignación retenga la integridad de cada cuenta vinculada aplicando _afinidad de la cuenta_. La afinidad garantiza que una cuenta que no usa un servicio específico no tenga asignado ningún costo de este servicio. Los costos acumulados en esa cuenta permanecen allí y las directivas de asignación no los calculan. Por ejemplo, podría tener cinco cuentas vinculadas. Si solo tres de ellas usan servicios de almacenamiento, el costo de los servicios de almacenamiento solo se asigna entre etiquetas en las tres cuentas.

Use el informe de análisis de costos para:

- Mostrar una vista agregada de toda su implementación durante un período de tiempo específico.
- Consultar los costos por categorías de etiquetas en función de las directivas creadas en el modelo de costos.

### <a name="cost-over-time-report"></a>Informe de costo a lo largo del tiempo

En el informe de costo a lo largo del tiempo se muestra el gasto a lo largo del tiempo, por lo que puede observar tendencias y detectar irregularidades en su implementación. En el se muestran básicamente los costos distribuidos a lo largo de un período definido. En el informe se muestran sus colaboradores de costos principales, incluidos los costos en curso y los cargos únicos de las instancias reservadas que se gastan durante un período de tiempo seleccionado. En este informe se usan las directivas establecidas en la [asignación de costos 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs).

Use el informe de costo a lo largo del tiempo para:

- Consultar los cambios a lo largo del tiempo y qué influencias cambian de un día (o intervalo de fechas) para otro.
- Analizar los costos a lo largo del tiempo para una instancia específica.
- Comprender por qué hubo un aumento del costo para una instancia específica.

### <a name="custom-charges-report"></a>Informe de gastos personalizados

Los usuarios de empresa y CSP a menudo tienen que proporcionar servicios agregados a sus clientes internos o externos, a lo que se une su propio consumo de recursos en la nube. Los cargos personalizados por servicios agregados o descuentos que se agregan a los informes de facturación o contracargo del cliente se definen como elementos de línea personalizados.

Los cargos de servicios personalizados reflejan los servicios que normalmente no se muestran en una factura. Los cargos personalizados que se crean se muestran entonces en informes de costos.

*Los cargos personalizados no son precios personalizados*. La lista de cargos personalizados no muestra las diferentes tasas que pueda estar cobrando. Por ejemplo, los cargos de facturación de AWS se muestran tal y como se cobran.

Para crear un cargo personalizado:

1. En **Custom Charges** (Cargos personalizados), haga clic en **Add New** (Agregar nuevo). Se muestra el cuadro de diálogo _Add New Custom Charge_ (Agregar nuevo cargo personalizado).
2. En **Provider Name** (Nombre del proveedor), escriba el nombre del proveedor.
3. En **Service Name** (Nombre del servicio), especifique el tipo de servicio.
4. En **Description** (Descripción), agregue una descripción para el cargo personalizado.
5. En **Type** (Tipo), seleccione **Percentage** (Porcentaje) y, en la lista desplegable de servicios, seleccione los servicios que se incluirán como cargos personalizados en los informes de costos.
6. En **Payment** (Pago), seleccione si el cargo es único o periódico. Si el cargo es periódico, seleccione Amortized (Amortizado) si quiere que el cargo se amortice y seleccione el número de meses.
7. En **Dates** (Fechas), si se selecciona un cargo único, en **Effective Date** (Fecha efectiva), escriba la fecha en que se paga el cargo. Si se selecciona el cargo periódico, escriba el intervalo de fechas con la fecha de inicio y la fecha de finalización del cargo.
8. En el **árbol de entidades**, seleccione las entidades a las que quiere que se aplique el cargo y, luego, seleccione **On** (Activado).

_Cuando los cargos se asignan a una entidad, los usuarios no pueden cambiarlos. Los cargos que agrega un administrador a una entidad principal son de solo lectura._

Para ver los cargos personalizados:

Los cargos personalizados se muestran en los informes de costos. Por ejemplo, abra el informe de análisis de costo real y, luego, en **Extended Filters** (Filtros extendidos), seleccione **Standalone** (Independiente). A continuación, el filtro muestra los **cargos personalizados**.

### <a name="cost-allocation-360"></a>Asignación de costos 360

Use la asignación de costos 360 para crear modelos de asignación de costos personalizados para asignar costos a los recursos en la nube consumidos. Muchos informes muestran información de modelos de costos personalizado que ha creado con modelos de costos personalizados. Y algunos informes solo muestran información después de haber creado un modelo de costos personalizados con asignación de costos.

Para más información sobre cómo crear modelos de costos personalizados, consulte [Tutorial: Administración de costos mediante Cloudyn](tutorial-manage-costs.md).

### <a name="cost-vs-budget-over-time-report"></a>Informe de costo frente a presupuesto a lo largo del tiempo

El informe de costo frente a presupuesto a lo largo del tiempo permite comparar los contribuyentes principales a los costos con el presupuesto. El presupuesto asignado aparece en el informe para que pueda ver el consumo del presupuesto (por encima/por debajo/normal) con el tiempo. Mediante la opción para mostrar u ocultar campos de la parte superior del informe, puede seleccionar ver el costo, el presupuesto, el costo acumulado y el presupuesto total.

### <a name="current-month-projected-cost-report"></a>Informe de costo previsto del mes en curso

El informe de costo previsto del mes en curso proporciona información detallada sobre el resumen del costo del mes en curso hasta la fecha. Este informe muestra los costos desde el principio del mes, del mes anterior, y el costo total previsto del mes en curso. El costo previsto del mes en curso se calcula como la suma del costo mensual actualizado y una previsión basada en el costo supervisado en los últimos 30 días.

Use el informe de costo previsto del mes en curso para:

- Proyectar los costos mensuales por servicio
- Proyectar los costos mensuales por cuenta

### <a name="annual-projected-cost-report"></a>Informe de costo previsto anual

El informe de costo previsto anual permite ver los costos previstos anuales en función de las tendencias de gasto anteriores. Muestra los costos previstos generales de los próximos 12 meses. Las previsiones se realizan mediante una función de tendencia extrapolada durante los 12 meses siguientes, en función de los costos asociados con los últimos 30 días de uso.

### <a name="budget-management-settings"></a>Configuración de administración de presupuesto

La administración de presupuesto le permite establecer un presupuesto para el año fiscal.

Para agregar un presupuesto a una entidad:

1. En la página Budget Management (Administración del presupuesto), en **Entidades**, seleccione la entidad en la que desea crear el presupuesto.
2. En el año del presupuesto, seleccione el año en el que desea crear el presupuesto.
3. En cada mes, establezca el presupuesto y, luego, haga clic en **Save** (Guardar).

Para importar un archivo para el presupuesto anual:

1. En **Acciones**, seleccione **Exportar** para exportar una plantilla CSV vacía y utilizarla como base para el presupuesto.
2. Complete el archivo CSV con las entradas del presupuesto y guárdelo localmente.
3. En **Acciones**, seleccione **Importar**.
4. Seleccione el archivo guardado y haga clic en **Ok** (Aceptar).

Para exportar el presupuesto completado como archivo CSV, en **Acciones**, seleccione **Exportar** para descargar el archivo.

Al finalizar, el presupuesto se muestra en los informes de análisis de costos y en el informe de de costo frente a presupuesto a lo largo del tiempo. También puede programar informes según umbrales de presupuesto.

### <a name="azure-resource-explorer-report"></a>Informe de Azure Resource Explorer

El informe de Azure Resource Explorer muestra una lista de todos los recursos de Azure disponibles en Cloudyn. Para usar eficazmente el informe, las cuentas de Azure deben tener habilitadas las métricas extendidas. Las métricas extendidas proporcionan a Cloudyn acceso a las máquinas virtuales de Azure. Para más información, consulte [Adición de métricas extendidas a máquinas virtuales de Azure](azure-vm-extended-metrics.md).

### <a name="azure-resources-over-time-report"></a>Informe de recursos de Azure a lo largo del tiempo

El informe de recursos de Azure a lo largo del tiempo muestra un desglose de todos los recursos que se ejecutan durante un período determinado. Para usar eficazmente el informe, las cuentas de Azure deben tener habilitadas las métricas extendidas. Las métricas extendidas proporcionan a Cloudyn acceso a las máquinas virtuales de Azure. Para más información, consulte [Adición de métricas extendidas a máquinas virtuales de Azure](azure-vm-extended-metrics.md).

### <a name="instance-explorer-report"></a>Informe de Explorador de instancias

El informe de Explorador de instancias se usa para ver diversas métricas de los recursos de las máquinas virtuales. Puede profundizar en instancias específicas para ver información como:
- Intervalos de ejecución de instancias
- Ciclo de vida en el período seleccionado
- Uso de CPU
- Entrada de red
- Tráfico de salida
- Discos activos

El informe de Explorador de instancias recopila todos los intervalos de ejecución dentro del intervalo de fechas definido y agrega datos en consecuencia. Para ver cada uno de los intervalos de ejecución durante el intervalo de fechas, expanda la instancia. El costo de cada instancia se calcula para el intervalo de fechas seleccionado en función de los precios de lista de AWS y Azure. No se aplican descuentos. Puede agregar campos adicionales al informe mediante la opción de mostrar u ocultar campos.

Use el informe de Explorador de instancias para:

- Calcular el costo estimado por máquina
- Crear una lista completa, con las horas de ejecución agregadas, de todas las máquinas que estaban activas durante un intervalo de tiempo
- Crear una lista por proveedor de servicios en la nube o cuenta
- Ver las máquinas creadas o terminadas durante un intervalo de tiempo
- Ver todas las máquinas actualmente detenidas
- Ver las etiquetas de cada máquina

### <a name="instances-over-time-report"></a>Informe de instancias a lo largo del tiempo

Con el informe de instancias a lo largo del tiempo, puede ver el número máximo de máquinas que estaban activas durante el intervalo de tiempo seleccionado. Si la resolución definida es por semana o mes, los resultados son el número máximo de máquinas activas en un día determinado durante ese mes. Seleccione un intervalo de fechas para seleccionar los filtros que quiere que se muestren en el informe.

### <a name="instance-utilization-over-time-report"></a>Informe de utilización de instancias a lo largo del tiempo

Este informe muestra un desglose del uso de CPU o memoria con el paso del tiempo para todas las instancias.

### <a name="compute-power-cost-over-time-report"></a>Informe de costo de potencia de proceso a lo largo del tiempo

El informe de costo de potencia de proceso a lo largo del tiempo proporciona un desglose de la potencia de proceso a lo largo de un intervalo de fechas especificado. Aunque otros informes muestran el número de máquinas en ejecución o las horas de tiempo de ejecución, este informe muestra las horas de núcleo, las horas por unidad de proceso o las horas de GB de RAM.

Use el informe para:

- Comprobar la potencia de proceso dentro de un intervalo de fechas especificado
- Ver los tiempos de proceso basados en los modelos de asignación de costos

Este informe está vinculado a las directivas de [asignación de costos 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs), por lo que los resultados se muestran según el etiquetado definido y las directivas de costo seleccionadas. Cuando no tiene creada una directiva, no se muestran resultados.

### <a name="compute-power-average-cost-over-time-report"></a>Informe de costo medio de potencia de proceso a lo largo del tiempo

El informe de costo medio de potencia de proceso a lo largo del tiempo se usa para ver algo más que el costo de cada máquina en ejecución. El informe muestra el costo medio por hora de la instancia, hora de núcleo, hora de unidad de proceso y hora de GB de RAM. El informe proporciona información detallada sobre la eficacia de la implementación.

Este informe está vinculado a las directivas de [asignación de costos 360](tutorial-manage-costs.md#use-custom-tags-to-allocate-costs), por lo que los resultados se muestran según el etiquetado definido y las directivas de costo seleccionadas. Cuando no tiene creada una directiva, no se muestran resultados.

### <a name="s3-cost-over-time-report"></a>Informe de costo a lo largo del tiempo de S3

El informe de costo a lo largo del tiempo de S3 proporciona un desglose de los costos de Amazon Simple Storage Service (S3) por cubo a lo largo del tiempo durante un período de tiempo especificado. El informe le ayuda a encontrar los cubos que son factores de costos principales, y muestra las tendencias en el uso y los gastos de S3.

### <a name="s3-distribution-of-cost-report"></a>Informe de distribución de costo de S3

Use el informe para analizar el costo de S3 del último mes por cubo y clase de almacenamiento. Puede usar la vista de gráfico circular para establecer el umbral de visibilidad. O bien, puede usar la vista de tabla para ver los subtotales.

### <a name="s3-bucket-properties-report"></a>Informe de propiedades de cubo de S3

Use el informe para ver las propiedades de cubo de S3. Puede usar la vista de gráfico circular para establecer el umbral de visibilidad. O bien, puede usar la vista de tabla para ver los subtotales.

### <a name="rds-instances-over-time-report"></a>Informe de instancias de RDS a lo largo del tiempo

Use el informe para ver un desglose de todas las instancias de Amazon Relational Database Service (RDS) en ejecución durante el período especificado.

### <a name="rds-active-instances-report"></a>Informe de instancias activas de RDS

Use el informe para analizar las instancias activas de RDS. En el informe, expanda el elemento de línea para ver información adicional.

### <a name="azure-reserved-instances-report"></a>Informe de instancias reservadas de Azure

El informe de instancias reservadas de Azure proporciona una vista única de todas las instancias reservadas de Azure. Este informe muestra cada compra en su propio artículo de línea. El informe también muestra detalles sobre dicha compra, como la cuenta con la que se compró, el tipo de compra y el tipo de instancia, los días restantes, etc. Puede mostrar u ocultar los datos del informe con la opción para mostrar u ocultar campos.

Use el informe de instancias reservadas de Azure para ver:

- Una lista de todas las reservas por fecha de compra.
- El tiempo restante hasta que expire la instancia reservada.
- Los cargos únicos.
- La cuenta con la que se compraron las instancias reservadas y cuándo.

### <a name="aws-reserved-instances-report"></a>Informe de instancias reservadas de AWS

El informe de instancias reservadas de AWS proporciona una vista única de todas las instancias reservadas de AWS. Este informe muestra cada compra en su propio elemento de línea y detalles sobre dicha compra, como la cuenta con la que se compró, el tipo de compra y el tipo de instancia, los días restantes, etc. Puede mostrar u ocultar los datos del informe con la opción para mostrar u ocultar campos.

Use el informe de instancias reservadas de AWS para ver:

- Una lista de todas las reservas por fecha de compra.
- El tiempo restante hasta que expire la instancia reservada.
- Los cargos únicos.
- El identificador original de compra (identificador de reserva).
- La cuenta con la que se compraron las instancias reservadas y cuándo.

### <a name="ec2-ri-buying-recommendations-report"></a>Informe de recomendaciones de compra de EC2

La base del consumo de recursos en la nube es el modelo a petición, donde los recursos generan un costo solo cuando se usan. No hay ningún compromiso por adelantado, solo se paga por lo que se usa y cuando se usa.

AWS ofrece una alternativa al modelo de precios de sus servicios Elastic Compute Cloud (EC2): la instancia reservada (RI). Este modelo de precios garantiza a los usuarios la capacidad siempre que la necesiten en el período de duración de la instancia reservada. La instancia reservada ofrece importantes descuentos sobre los precios a petición. A cambio, los usuarios se comprometen por adelantado a usar una instancia virtual. El compromiso está vinculado a una familia, tamaño, zona de disponibilidad (AZ) y sistema operativo concretos, durante el período de compromiso (uno o tres años). La instancia reservada permite que AWS planee con eficiencia la capacidad futura, así como obtener el compromiso del cliente al uso de sus servicios.

Tres opciones de pago para instancias reservadas, que son:

- Todo por adelantado: suma masiva el día 0, que ofrece el mayor descuento.
- No por adelantado: donde el costo de la instancia reservada se paga en plazos mensuales lo que dura la instancia reservada. Esta opción ofrece el descuento más bajo.
- Parte por adelantado: donde la mitad o la cuarta parte del precio se paga por adelantado, y el resto en plazos mensuales, con una tasa de descuento inferior, pero cercana, a la tasa de todo por adelantado.

Cloudyn evalúa el tiempo de actividad de cada máquina durante los últimos 30 días. Cloudyn recomienda comprar instancias administradas cuando resulte más rentable ejecutar la máquina con una instancia administrada en el nivel de tiempo de actividad actual.

El informe muestra la justificación de sus recomendaciones para ahorrar la mayor cantidad de dinero posible durante el año. Las recomendaciones sugieren reemplazar las instancias a petición por instancias reservadas. Puede comprar instancias reservadas directamente desde el informe.

Cada pestaña se abre como un informe completo. Las secciones importantes de las pestañas son:

- **EC2 RI Purchase Impact** (Impacto de compra de instancia reservada para EC2): esta sección proporciona una simulación de la diferencia entre instancias a petición y reservadas. Haga clic en **Zoom in** (Acercar) para ver el informe completo de impacto de compra de instancia reservada para EC2 con los filtros ya definidos para su recomendación. Este informe muestra el impacto de compra de todas las posibles compras de instancia reservada. Puede ajustar el tiempo de actividad medio esperado para ver la posibilidad de ahorrar al comprar instancias reservadas de EC2.

- **Saving Analysis** (Análisis de ahorros): esta sección proporciona los posibles ahorros conseguidos y el mes en que se actualizan los ahorros cuando se siguen las recomendaciones de Cloudyn. Los ahorros reales y el porcentaje ahorrado se resaltan en rojo.

- **EC2 RI Type Comparison** (Comparación de tipo de instancia reservada para EC2): esta sección enfatiza los puntos destacables del ROI de la implementación recomendada de Cloudyn, incluidas todas las opciones pertinentes. Los resultados de este informe dan por hecho que la máquina se ejecuta con un tiempo de actividad del 100 %. Haga clic en **Zoom In** (Acercar) para abrir el informe detallado.

- **Instances Over Time** (Instancias a lo largo del tiempo): esta sección muestra un desglose de todas las instancias asociadas con la recomendación: a petición, instancias reservadas y de prioridad baja. Haga clic en **Zoom In** (Acercar) para abrir el informe detallado.
- **Breakeven Points** (Puntos de equilibrio): esta sección muestra una tabla de todas las posibles implementaciones recomendadas, junto con el ROI y el mes en que este se produce. Haga clic en **Zoom In** (Acercar) para abrir el informe detallado.

### <a name="ec2-reservations-over-time-report"></a>Informe de reservas de EC2 a lo largo del tiempo

El informe de reservas de EC2 a lo largo del tiempo realiza un seguimiento del estado de uso de las instancias reservadas de EC2 compradas. Puede establecer la resolución del informe en hora, día o semana.

Use el informe para:

- Mostrar las reservas compradas usadas y no usadas
- Explorar en profundidad la resolución por hora para ver el uso de la instancia reservada por hora

### <a name="savings-over-time-report"></a>Informe de ahorros a lo largo del tiempo

Use el informe de ahorros a lo largo del tiempo para ver los ahorros conseguidos mediante instancias reservadas e instancias de prioridad baja. El informe muestra el ROI conseguido a lo largo del tiempo resultante de las compras de instancias reservadas.

Para ver los ahorros de las instancias reservadas, agrupe los resultados por **modelo de precios** y seleccione **Reservation** (Reserva). Para ver los ahorros de instancias reservadas conseguidos por una cuenta o tipo de instancia en concreto, agregue la agrupación pertinente y filtre por la cuenta o el tipo de instancia.

Para ver los ahorros derivados del uso de instancias de prioridad baja, filtre el **modelo de precios** por **Spot** (Prioridad baja). El filtro predeterminado de este informe es instancias reservadas e instancias de prioridad baja.

### <a name="rds-ri-buying-recommendations-report"></a>Informe de recomendaciones de compra de instancias reservadas para RDS

El informe de recomendaciones de compra de instancias reservadas para RDS recomienda cuándo usar instancias reservadas de RDS en lugar de instancias a petición.

Cada pestaña se abre como un informe completo. Las secciones importantes de las pestañas son:

- **RDS RI Purchase Impact** (Impacto de compra de instancia reservada para RDS): esta sección proporciona una simulación de la diferencia entre las instancias a petición y reservadas. Haga clic en **Zoom in** (Acercar) para ver el informe completo de impacto de compra de instancia reservada para RDS con los filtros ya definidos para su recomendación. Este informe permite ver el impacto de compra de todas las posibles compras de instancias reservadas.  Puede ajustar el tiempo de actividad medio esperado y ver los posibles ahorros con la compra de instancias reservadas.
- **Saving Analysis** (Análisis de ahorros): esta sección proporciona los posibles ahorros conseguidos y el mes en que se actualizan los ahorros cuando se siguen las recomendaciones de Cloudyn. Los ahorros reales y el porcentaje ahorrado se resaltan en rojo.

- **RDS RI Type Comparison** (Comparación de tipo de instancia reservada para RDS): esta sección enfatiza los puntos destacables del ROI de la implementación recomendada, incluidas todas las opciones pertinentes. Los resultados de este informe dan por hecho que la máquina se ejecuta con un tiempo de actividad del 100 %. Haga clic en **Zoom In** (Acercar) para abrir el informe detallado de la máquina seleccionada.
- **Instances Over Time** (Instancias a lo largo del tiempo): esta sección muestra un desglose de todas las instancias asociadas con la recomendación: a petición, instancias reservadas y de prioridad baja. Haga clic en **Zoom In** (Acercar) para abrir el informe detallado.

- **Breakeven Points** (Puntos de equilibrio): esta sección muestra una tabla de todas las posibles implementaciones recomendadas, junto con el ROI y el mes en que este se produce. Haga clic en **Zoom In** (Acercar) para abrir el informe detallado.

### <a name="rds-reservations-over-time-report"></a>Informe de reservas de RDS a lo largo del tiempo

Use el informe de reservas de RDS a lo largo del tiempo para ver un desglose de sus reservas usadas y no usadas durante el período especificado.

### <a name="reserved-instance-purchase-impact-report"></a>Informe de impacto de compra de instancias reservadas

El informe de impacto de compra de instancias reservadas de EC2 permite simular el costo de las instancias reservadas frente al costo a petición a lo largo del tiempo. Puede ayudarle a tomar mejores decisiones de compra. Ajuste los filtros, como el tiempo de ejecución medio, los plazos, la plataforma, etc., para tomar decisiones argumentadas al considerar las compras de instancias reservadas.

### <a name="cost-effective-sizing-recommendations-report"></a>Informe de recomendaciones de tamaño rentable

El informe de recomendaciones de tamaño rentable proporciona resultados para Azure y AWS. Para los usuarios de AWS, se tienen en cuenta las compras de instancias reservadas y los resultados no incluyen las máquinas que se ejecutan como instancias reservadas. Este informe proporciona una lista de instancias infrautilizadas que son candidatas a la reducción de tamaño. Las recomendaciones se basan en los datos de uso y rendimiento de los últimos 30 días. En cada recomendación hay una lista de candidatas a reducción de tamaño, la justificación de la reducción de tamaño y un vínculo para ver detalles completos y métricas de rendimiento de la instancia. Además, se indica cuándo las recomendaciones pertinentes aconsejan cambiar a tipos de instancia de una generación más reciente.

No se puede descargar de este informe la lista de identificadores de instancia para los que se recomienda reducir el tamaño. Para descargar los identificadores de instancia, use el informe de recomendaciones de todos los tamaños.

Considere el siguiente ejemplo de reducción de tamaño:

Tiene seis instancias de ejecución m3.xlarge. El análisis de Cloudyn muestra que cinco de ellas tienen una baja utilización de CPU. Considere la posibilidad de reducir el tamaño de ellas.

En la sección Cost Impact (Efecto del costo), se calcula el efecto del costo. En este ejemplo, al expandir el elemento de línea, puede ver que el precio actual de una instancia de m3.xlarge (Linux/Unix) es de 0,266 $por hora y que el de una instancia m3.large (Linux/Unix) es de 0,133 $ por hora. Por lo tanto, el costo anual es de 11,651 $ por cinco instancias m3.xlarge que se ejecutan al 100 % de utilización. El costo anual es de 5,825 $ por cinco instancias m3.large que se ejecutan al 100 % de utilización. Los ahorros potenciales son de 5,825 $.

Para ver las justificaciones de tamaño rentable, haga clic en + para expandir el elemento de línea. En **Details** (Detalles):

- La sección **Recommendation Justification** (Justificación de recomendación) muestra la implementación actual y el número de instancias para las que se recomienda reducir su tamaño.
- La sección **Cost Impact** (Efecto del costo) muestra el cálculo usado para determinar los ahorros potenciales.
- La sección **Potential Annual Savings** (Ahorros potenciales anuales) muestra los ahorros potenciales anuales al reducir de tamaño según cada recomendación de Cloudyn.

### <a name="all-sizing-recommendations-report"></a>Informe de recomendaciones de todos los tamaños

Este informe proporciona una lista de instancias infrautilizadas que son candidatas a la reducción de tamaño. Las recomendaciones se basan en los datos de uso y rendimiento de los últimos 30 días. En cada recomendación, puede ver detalles completos y métricas de rendimiento de la instancia.

Si ha comprado instancias reservadas de AWS, este informe contiene los resultados de todas las instancias en ejecución, incluidas las instancias que se ejecutan como instancias reservadas.

Use el informe de recomendaciones de todos los tamaños para:

- Ver una lista de todas las instancias que son candidatas a la reducción de tamaño.
- Exportar una lista de informes que contiene los nombres y los identificadores de instancia.

Para ver los detalles de recomendación de una instancia concreta, haga clic en **+** para ampliar los detalles. La sección de detalles de recomendación proporciona información general de la recomendación.

La sección **Tags** (Etiquetas) proporciona la lista de las claves y los valores de etiqueta de la instancia seleccionada. Use las etiquetas del panel izquierdo para filtrar la sección.

La sección **CPU Utilization** (Utilización de CPU) proporciona la utilización de CPU de la instancia durante el último mes, cada día.

Haga clic en el gráfico para explorarlo en profundidad y abra el informe de CPU de instancias a lo largo del tiempo para ver un desglose de las instancias.

- Use la opción **Show/Hide Fields** (Mostrar u ocultar campos) para agregar o quitar campos: Timestamp, Avg CPU, Min CPU, Max CPU.
- Use **Date Range** (Intervalo de fechas) para escribir una fecha o un intervalo de fechas y profundizar en un identificador de instancia específico.
- Use **Extended Filters** (Filtros extendidos) para mostrar todos los identificadores de instancia o uno específico.
- Haga clic en **Zoom in** (Acercar) para abrir el informe de utilización de CPU.

Si la instancia no se han supervisado durante 30 días, se muestran datos incompletos.

La sección **Memory Utilization (GB)** (Utilización de memoria [GB]) proporciona información sobre la memoria utilizada. Los usuarios de AWS no tienen disponibles las métricas de memoria automáticamente y se deben agregar por instancia a través de AWS. AWS cobra por habilitar las métricas de memoria de las instancias de EC2.

La sección **Memory Utilization (%)** (Utilización de memoria [%]) muestra el porcentaje de memoria usada.

La sección **Network Input Traffic** (Tráfico de entrada de red) muestra una instantánea a lo largo del tiempo del tráfico de red, medio y máximo, de la instancia seleccionada. Mantenga el mouse sobre las líneas para ver la fecha y el tráfico máximo en ese momento. Haga clic en **Zoom In** (Acercar) para abrir el informe de tráfico de entrada de red.

La sección **Network Output Traffic** (Tráfico de salida de red) muestra una instantánea del tráfico de salida de red de la instancia seleccionada. Mantenga el mouse sobre las líneas para ver la fecha y el tráfico máximo en ese momento. Haga clic en **Zoom In** (Acercar) para abrir el informe de tráfico de salida de red.

### <a name="instance-metrics-explorer-report"></a>Informe de Explorador de métricas de instancia

El informe de Explorador de métricas de instancia muestra las métricas de rendimiento de toda la nube por cada instancia. Use el informe para ver las instancias que se utilizan en exceso o que están infrautilizados según los umbrales de métricas de CPU, memoria y red.

Para ver el rendimiento de toda la nube por cada instancia:

1. En **Date Range** (Intervalo de fechas), seleccione el intervalo de fechas para el que quiere ver el rendimiento.
2. En **Tags** (Etiquetas) seleccione las etiquetas que quiere ver.
3. En **Filters** (Filtros), seleccione los filtros que quiere mostrar en el informe.
4. En **Extended Filters** (Filtros extendidos), ajuste los umbrales del informe para:
    - Uso medio de CPU
    - Uso máximo de CPU
    - Uso medio de memoria
    - Uso máximo de memoria
5. En **Extended Filters** (Filtros extendidos), haga clic en **Show** (Mostrar) y luego seleccione el tipo de instancias para mostrar.

Para ver las métricas de una instancia específica a lo largo del tiempo:

- En el informe de Explorador de métricas de instancia, haga clic en **+** para ver los detalles.

### <a name="rds-sizing-recommendations-report"></a>Informe de recomendaciones de tamaño de RDS

El informe de recomendaciones de tamaño de RDS proporciona recomendaciones de tamaño de RDS para optimizar el uso de la nube. Este informe proporciona una lista de instancias infrautilizadas que son candidatas a la reducción de tamaño. Las recomendaciones de Cloudyn se basan en los datos de uso y rendimiento de los últimos 30 días. Puede filtrar las recomendaciones por nombre de cuenta, región, tipo de instancia y estado.

### <a name="sizing-threshold-manager-report"></a>Informe de administrador de umbrales de tamaño

Las recomendaciones de tamaño integradas de Cloudyn se calculan mediante un complejo algoritmo para proporcionar sugerencias de tamaño precisas. Puede ajustar los umbrales para las recomendaciones de reducción de tamaño.

Para ajustar manualmente las recomendaciones de umbral de tamaño:

1. En el administrador de umbrales de tamaño, ajuste los umbrales siguientes según sus preferencias:
    - % uso medio de CPU
    - % uso máximo de CPU
    - % uso medio de memoria
    - % uso máximo de memoria
3. Haga clic en **Apply** (Aplicar) para guardar los cambios.
4. Los cambios se aplican inmediatamente a todas las recomendaciones.

Para restaurar los umbrales predeterminados:

- En el administrador de umbrales de tamaño, haga clic en **Restore Defaults** (Restaurar valores predeterminados).

### <a name="compute-instance-types-report"></a>Informe de tipos de instancias de proceso

Use el informe de tipos de instancia para:

- Ver los tipos de instancia por servicio, familia, nombre de API y nombre.
- Ver detalles tales como CPU, ECU, RAM y red.

Puede usar **Search** (Buscar) para buscar elementos de línea específicos.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo usar los informes, por ejemplo, cómo personalizarlos o guardarlos y programarlos, consulte [Informes de costos](understanding-cost-reports.md).
- Para información sobre los paneles incluidos en Cloudyn y sobre cómo crear sus propios paneles personalizados, consulte [Vista de las métricas clave de costos con paneles](dashboards.md).
