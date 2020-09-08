---
title: Planeamiento y administración de costos de Azure Blob Storage
description: Aprenda a planear y administrar los costos de Azure Blob Storage mediante el análisis de costos en Azure Portal.
services: storage
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 08/03/2020
ms.author: normesta
ms.subservice: common
ms.custom: subject-cost-optimization
ms.openlocfilehash: 75b464c140bfda6c3f3559d3bfdbe1e6bc2e7f24
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760743"
---
# <a name="plan-and-manage-costs-for-azure-blob-storage"></a>Planeamiento y administración de costos de Azure Blob Storage

Este artículo le ayuda a planear y administrar los costos de Azure Blob Storage. En primer lugar, use la calculadora de precios de Azure para calcular los costos. Tras crear una cuenta de almacenamiento, optimícela para pagar solo lo que necesite. Use las características de la administración de costos para crear presupuestos y supervisar costos. También puede revisar los costos previstos y supervisar las tendencias de gasto para identificar las áreas en las que podría querer actuar.

Tenga en cuenta que los costos de Blob Storage son solo una parte de los costos mensuales de la factura de Azure. Aunque en este artículo se explica cómo calcular y administrar los costos de Blob Storage, se le facturarán todos los servicios y recursos de Azure que use para su suscripción de Azure, incluidos los servicios de terceros. Una vez que esté familiarizado con la administración de los costos de Blob Storage, puede aplicar métodos similares para administrar los costos de todos los servicios de Azure que se usan en su suscripción.

## <a name="estimate-costs"></a>cálculo de los costos

Use la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular los costos antes de crear y comenzar a transferir datos a una cuenta de Azure Storage.

1. En la página [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/), elija el icono **Cuentas de almacenamiento**.

2. Desplácese hacia abajo en la página y busque la sección **Cuentas de almacenamiento** de la estimación.

3. Elija las opciones de las listas desplegables. 

   A medida que se modifica el valor de estas listas desplegables, cambia el costo estimado. Esa estimación aparece en la esquina superior, así como en la parte inferior de la estimación. 
    
   ![Supervisión de costos con el panel Análisis de costos](media/storage-plan-manage-costs/price-calculator-storage-type.png)

   A medida que cambia el valor de la lista desplegable **Tipo**, también cambian otras opciones que aparecen en esta hoja de cálculo. Use los vínculos de la sección **Más información** para obtener más información sobre el significado de cada opción y cómo afectan al precio de las operaciones relacionadas con el almacenamiento. 

4. Modifique las demás opciones cómo afectan a la estimación.

## <a name="optimize-costs"></a>Optimización de costos

Considere la posibilidad de usar estas opciones para reducir costos. 

- Reservar capacidad de almacenamiento.

- Organizar los datos en capas de almacenamiento.

- Mover automáticamente los datos entre las capas de almacenamiento.

En esta sección se tratan todas las opciones de forma más detallada. 

#### <a name="reserve-storage-capacity"></a>Reserva de la capacidad de almacenamiento

Puede ahorrar dinero en costos de almacenamiento de datos de blobs con la capacidad reservada de Azure Storage. La capacidad reservada de Azure Storage ofrece un descuento en la capacidad para los blobs en bloques y los datos de Azure Data Lake Storage Gen2 en las cuentas de almacenamiento estándar cuando se compromete a una reserva durante un año o tres años. Una reserva proporciona una cantidad fija de capacidad de almacenamiento para el plazo de la reserva. La capacidad reservada de Azure Storage puede disminuir considerablemente los costos de capacidad de los blobs en bloques y los datos de Azure Data Lake Storage Gen2. 

Para más información, consulte [Optimización de los costos de Blob Storage con capacidad reservada](https://docs.microsoft.com/azure/storage/blobs/storage-blob-reserved-capacity).

#### <a name="organize-data-into-access-tiers"></a>Organización de datos en capas de almacenamiento

Para reducir los costos puede colocar los datos del blob en las capas de almacenamiento más económicas. Elija una de las tres capas que están diseñadas para optimizar los costos del uso de datos. Por ejemplo, el nivel de *acceso frecuente* tiene un mayor costo de almacenamiento, pero un menor costo de acceso. Por consiguiente, si planea acceder a los datos con frecuencia, es posible que este nivel sea la opción más económica. Si planea acceder a los datos con poca frecuencia, tendría más sentido elegir los niveles de acceso *esporádico* o *de archivo*, ya que su costo de acceso a los datos es mayor, pero se reduce el costo de almacenamiento.    

Para más información, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](https://docs.microsoft.com/azure/storage/blobs/storage-blob-storage-tiers?tabs=azure-portal).

#### <a name="automatically-move-data-between-access-tiers"></a>Movimiento automático de datos entre las capas de almacenamiento

Use las directivas de administración del ciclo de vida para mover periódicamente datos entre las distintas capas para ahorrar el mayor dinero posible. Para mover datos, estas directivas usan las reglas que especifique. Por ejemplo, puede crear una regla que mueva blobs al nivel de almacenamiento de archivo si el blob no se ha modificado en 90 días. La creación de directivas que ajusten el nivel de acceso de los datos le permite diseñar las opciones de almacenamiento más baratas que se ajusten a sus necesidades.

Para más información, consulte [Administración del ciclo de vida de Azure Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts?tabs=azure-portal)

## <a name="create-budgets"></a>Creación de presupuestos

Puede crear [presupuestos](../../cost-management-billing/costs/tutorial-acm-create-budgets.md) para administrar los costos y crear alertas que envíen notificaciones automáticamente a las partes interesadas sobre anomalías en los gastos y riesgos de gastos adicionales. Las alertas se basan en el gasto comparado con los umbrales de presupuesto y costo. Los presupuestos y las alertas se crean para las suscripciones y los grupos de recursos de Azure, por lo que son útiles como parte de una estrategia general de supervisión de costos. Sin embargo, pueden tener una funcionalidad limitada para administrar los costos de servicios individuales de Azure, como el costo de Azure Storage, porque están diseñados para realizar un seguimiento de los costos en un nivel más alto.

## <a name="monitor-costs"></a>Supervisión de costos

A medida que se usan recursos con Azure Storage, se incurre en costos. Los costos de la unidad de uso de recursos varían en función de intervalos de tiempo (segundos, minutos, horas y días) o en función del uso de unidades (bytes, megabytes, etc.). Tan pronto como empieza el uso de Azure Storage, se incurre en costos. Puede ver los costos en el panel [Análisis de costos](../../cost-management-billing/costs/quick-acm-cost-analysis.md) de Azure Portal.

Al usar el análisis de costos, puede ver los costos de Azure Storage en gráficos y tablas para diferentes intervalos de tiempo. Algunos ejemplos son: por día, mes actual y anterior y año. También puede ver los costos comparados con los presupuestos y los costos previstos. Cambiar a vistas más largas en el tiempo puede ayudarle a identificar las tendencias de gasto y ver dónde podría haber ocurrido un gasto excesivo. Si ha creado presupuestos, también podrá ver fácilmente dónde se han excedido.

>[!NOTE]
> El análisis de costos es compatible con varios tipos de cuenta de Azure. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](../../cost-management-billing/costs/understand-cost-mgt-data.md) (Información sobre los datos de Cost Management). Para ver los datos de costos, se necesita al menos acceso de lectura en la cuenta de Azure. Para más información acerca de cómo asignar acceso a los datos de Azure Cost Management, consulte [Asignación de acceso a los datos](../../cost-management-billing/costs/assign-access-acm-data.md).

Para ver los costos de Azure Storage en los análisis de costos:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).

2. Abra la ventana **Administración de costos y facturación**, seleccione **Administración de costos** en el menú y, a continuación, seleccione **Análisis de costos**. A continuación, puede cambiar el ámbito a una suscripción específica en la lista desplegable **Ámbito**.

   ![Supervisión de costos con el panel Análisis de costos](./media/storage-plan-manage-costs/cost-analysis-pane.png)

4. Para ver los costos de Azure Storage, seleccione **Agregar filtro** y, a continuación, seleccione **Nombre del servicio**. Después, elija **almacenamiento** en la lista. 

   Este es un ejemplo que muestra solo los costos de Azure Storage:

   ![Supervisión de costos de almacenamiento con el panel Análisis de costos](./media/storage-plan-manage-costs/cost-analysis-pane-storage.png)

En el ejemplo anterior, hemos visto el costo actual del servicio. También se muestran los costos por regiones de Azure (ubicaciones) y por grupo de recursos. También puede agregar otros filtros (por ejemplo, un filtro para ver los costos de cuentas de almacenamiento concretas).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la administración de costos con los [análisis de costos](../../cost-management-billing/costs/quick-acm-cost-analysis.md).

Consulte los siguientes artículos para obtener información sobre cómo funcionan los precios con Azure Storage:

- [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/)
- [Optimización de los costos de almacenamiento de blobs con capacidad reservada](../blobs/storage-blob-reserved-capacity.md)
