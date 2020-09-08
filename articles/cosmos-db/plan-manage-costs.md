---
title: Planeamiento y administración de los costos de Azure Cosmos DB
description: Obtenga información acerca de cómo planear y administrar los costos de Azure Cosmos DB mediante el análisis de costos en Azure Portal.
author: SnehaGunda
ms.author: sngun
ms.custom: subject-cost-optimization
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: 7f0a8fcb841399eb910f5f043cc75ddad037ee30
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88606864"
---
# <a name="plan-and-manage-costs-for-azure-cosmos-db"></a>Planeamiento y administración de los costos de Azure Cosmos DB

En este artículo se describe cómo puede planear y administrar los costos de Azure Cosmos DB:

- Calcular cuál será el costo antes de crear recursos
- Revisar los costos estimados al empezar a usar los recursos
- Usar las características de la administración de costos para crear presupuestos y supervisar costos
- Revisar los costos previstos e identificar las tendencias de gasto para determinar las áreas interesantes para actuar

Tenga en cuenta que los costos de Azure Cosmos DB son solo una parte de los costos mensuales de la factura de Azure. Si usa otros servicios de Azure, se le facturarán todos los servicios y recursos de Azure usados en la suscripción de Azure, incluidos los servicios de terceros. En este artículo se explica cómo planear y administrar los costos de Azure Cosmos DB. Una vez que esté familiarizado con la administración de los costos de Azure Cosmos DB, puede aplicar métodos similares para administrar los costos de todos los servicios de Azure que se usan en la suscripción.

## <a name="prerequisites"></a>Requisitos previos

El análisis de costos es compatible con varios tipos de cuenta de Azure. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../cost-management-billing/costs/assign-access-acm-data.md).

## <a name="provisioned-throughput-or-serverless"></a>Rendimiento aprovisionado o sin servidor

Azure Cosmos DB admite dos tipos de modos de capacidad: [rendimiento aprovisionado](set-throughput.md) y [sin servidor](serverless.md). La forma de cobrar por el uso de Azure Cosmos DB varía mucho entre estos dos modos, por lo que es importante elegir el que mejor se adapte a su carga de trabajo. Consulte el artículo sobre [cómo elegir entre el modo de rendimiento aprovisionado y el modo sin servidor](throughput-serverless.md) para obtener instrucciones y recomendaciones sobre cómo tomar esta decisión.

## <a name="estimating-provisioned-throughput-costs-with-capacity-calculator"></a>Cálculo de los costos de rendimiento aprovisionado con la calculadora de capacidad

Si tiene previsto usar Azure Cosmos DB en el modo de rendimiento aprovisionado, use la [calculadora de capacidad de Azure Cosmos DB](https://cosmos.azure.com/capacitycalculator/) para calcular los costos antes de crear los recursos en una cuenta de Azure Cosmos. La calculadora de capacidad se usa para obtener una estimación del rendimiento necesario y el costo de la carga de trabajo. La configuración de las bases de datos y los contenedores de Azure Cosmos con la cantidad adecuada de rendimiento aprovisionado, o [unidades de solicitud (RU/s)](request-units.md), para la carga de trabajo es esencial para optimizar el costo y el rendimiento. Tiene que escribir detalles como el tipo de API, el número de regiones, el tamaño de los elementos, las solicitudes de lectura/escritura por segundo y el total de datos almacenados para obtener una estimación de costos. Para más información acerca de la calculadora de capacidad, consulte el artículo [Estimación](estimate-ru-with-capacity-planner.md).

En la captura de pantalla siguiente se muestra el rendimiento y la estimación de costos mediante la calculadora de capacidad:

:::image type="content" source="./media/plan-manage-costs/capacity-calculator-cost-estimate.png" alt-text="Estimación de costos en la calculadora de capacidad de Azure Cosmos DB":::

## <a name="estimating-serverless-costs"></a>Evaluación de los costos sin servidor

Si tiene previsto usar Azure Cosmos DB en el modo sin servidor, debe calcular el número de [unidades de solicitud](request-units.md) y GB de almacenamiento que puede consumir mensualmente. Puede calcular la cantidad de unidades de solicitud necesaria mediante la evaluación del número de operaciones de base de datos que se emitirán en un mes y la multiplicación de su importe por el costo de unidades de solicitud correspondientes. En la tabla siguiente se enumeran los cargos de unidades de solicitud estimados para las operaciones de base de datos comunes:

| Operación | Costo estimado | Notas |
| --- | --- | --- |
| Crear un elemento | 5 RU | Costo medio de un elemento de 1 KB con menos de cinco propiedades para indexar |
| Actualizar un elemento | 10 RU | Costo medio de un elemento de 1 KB con menos de cinco propiedades para indexar |
| Lectura de un elemento individual por su identificador y la clave de partición (lectura de punto) | 1 RU | Costo medio de un elemento de 1 KB |
| Eliminación de un elemento | 5 RU | |
| Ejecutar una consulta | 10 RU | Costo medio de una consulta que saca el máximo partido de la [indexación](index-overview.md) y devuelve 100 resultados o menos. |

> [!IMPORTANT] 
> Preste atención a las notas de la tabla anterior. Para una estimación más precisa de los costos reales de las operaciones, puede usar el [emulador de Azure Cosmos](local-emulator.md) y [medir el costo de unidades de solicitud exacto de las operaciones](find-request-unit-charge.md). Aunque el emulador de Azure Cosmos no admite el uso de servidores, informa de un cargo estándar de unidades de solicitud para las operaciones de base de datos y se puede usar para esta estimación.

Una vez que haya calculado el número total de unidades de solicitud y los GB de almacenamiento que probablemente consumirá durante un mes, la siguiente fórmula devolverá la estimación de costos: **([Número de unidades de solicitud]/1 millón * 0,25 USD) + ([GB de almacenamiento] * 0,25 USD)** .

> [!NOTE]
> Los costos que se muestran en el ejemplo anterior son solo para fines de demostración. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener la información más reciente sobre los precios.

## <a name="review-estimated-costs-from-the-azure-portal"></a>Revisión de los costos estimados en Azure Portal

Al comenzar a usar los recursos de Azure Cosmos DB desde Azure Portal, puede ver los costos estimados. Siga estos pasos para revisar la estimación de costos:

1. Inicie sesión en Azure Portal y vaya a la cuenta de Azure Cosmos.
1. Vaya a la sección de **información general**.
1. Compruebe el gráfico de **costos** en la parte inferior. En este gráfico se muestra una estimación del costo actual durante un período de tiempo configurable:
1. Cree un nuevo contenedor; por ejemplo, un contenedor de gráficos.
1. Especifique el rendimiento necesario para la carga de trabajo; por ejemplo, 400 RU/s. Después de especificar el valor del rendimiento, puede ver la estimación de precios, tal como se muestra en la siguiente captura de pantalla:

   :::image type="content" source="./media/plan-manage-costs/cost-estimate-portal.png" alt-text="Estimación de costos en Azure Portal":::

## <a name="use-budgets-and-cost-alerts"></a>Uso de presupuestos y alertas de costos

Puede crear [presupuestos](../cost-management/tutorial-acm-create-budgets.md) para administrar los costos y crear alertas que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. Sin embargo, pueden tener una funcionalidad limitada para administrar los costos de servicios individuales de Azure, como el costo de Azure Cosmos DB, porque están diseñados para realizar un seguimiento de los costos en un nivel más alto.

Si la suscripción de Azure tiene un límite de gasto, Azure le impide gastar por encima del importe del crédito. A medida que crea y usa recursos de Azure, se usan los créditos. Cuando alcance el límite de crédito, los recursos que ha implementado se deshabilitan durante el resto de ese período de facturación. No se puede cambiar el límite de crédito, pero se puede eliminar. Para más información sobre los límites de gasto, consulte [Límite de gasto de Azure](../billing/billing-spending-limit.md).

## <a name="monitor-costs"></a>Supervisión de costos

A medida que se usan recursos con Azure Cosmos DB, se incurre en costos. Los costos de la unidad de uso de recursos varían en función de intervalos de tiempo (segundos, minutos, horas y días) o en función del uso de unidades de solicitud. En cuanto se inicia el uso de Azure Cosmos DB, se incurre en costos y puede verlos en el panel [Análisis de costos](../cost-management/quick-acm-cost-analysis.md) de Azure Portal.

Al usar el análisis de costos, puede ver los costos de Azure Cosmos DB en gráficos y tablas para diferentes intervalos de tiempo. Algunos ejemplos son: por día, actual, mes anterior y año. También puede ver los costos comparados con los presupuestos y los costos previstos. Cambiar a vistas más largas en el tiempo puede ayudarle a identificar las tendencias de gasto y ver dónde podría haber ocurrido un gasto excesivo. Si ha creado presupuestos, también puede ver fácilmente dónde se han superado. Para ver los costos de Azure Cosmos DB en el análisis de costos:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).

1. Abra la ventana **Administración de costos y facturación**, seleccione **Administración de costos** en el menú y, a continuación, seleccione **Análisis de costos**. A continuación, puede cambiar el ámbito a una suscripción específica en la lista desplegable **Ámbito**.

1. De forma predeterminada, el costo de todos los servicios se muestra en el primer gráfico de anillos. Seleccione el área del gráfico con la etiqueta "Azure Cosmos DB".

1. Para restringir los costos de un único servicio, como Azure Cosmos DB, seleccione **Agregar filtro** y, a continuación, seleccione **Nombre del servicio**. A continuación, elija **Azure Cosmos DB** en la lista. Este es un ejemplo que muestra solo los costos de Azure Cosmos DB:
 
   :::image type="content" source="./media/plan-manage-costs/cost-analysis-pane.png" alt-text="Supervisión de costos con el panel Análisis de costos":::

En el ejemplo anterior, hemos visto el costo actual de Azure Cosmos DB del mes de febrero. Los gráficos también contienen los costos de Azure Cosmos DB por ubicación y por grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener información sobre cómo funcionan los precios en Azure Cosmos DB:

* [Modelo de precios de Azure Cosmos DB](how-pricing-works.md)
* [Optimización del costo de rendimiento aprovisionado en Azure Cosmos DB](optimize-cost-throughput.md)
* [Optimización de los costos de consulta de Azure Cosmos DB](optimize-cost-queries.md)
* [Optimización de los costos de almacenamiento de Azure Cosmos DB](optimize-cost-storage.md)