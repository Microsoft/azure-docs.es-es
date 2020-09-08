---
title: Ajuste de la capacidad para cargas de trabajo de consultas e índices
titleSuffix: Azure Cognitive Search
description: Ajuste los recursos de proceso de réplica y partición en Azure Cognitive Search, donde el precio de cada recurso se basa en unidades de búsqueda facturables.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/30/2020
ms.openlocfilehash: 476af7dd40cd1f31d03f3bd80affac0ce10ef900
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88927211"
---
# <a name="adjust-capacity-in-azure-cognitive-search"></a>Ajuste de la capacidad en Azure Cognitive Search

Antes de [aprovisionar un servicio de búsqueda](search-create-service-portal.md) y de adquirir un plan de tarifa específico, dedique unos minutos a comprender el papel de las réplicas y particiones en un servicio y cómo puede ajustar un servicio para acomodar los picos y caídas que se producen en la demanda de recursos.

La capacidad depende del [plan que elija](search-sku-tier.md) (los planes determinan las características del hardware) y la combinación de réplicas y particiones necesaria para las cargas de trabajo proyectadas. Según el plan y el tamaño del ajuste, el aumento o reducción de la capacidad puede tardar desde 15 minutos a varias horas. 

Al modificar la asignación de réplicas y particiones, se recomienda usar Azure Portal. El portal aplica límites a las combinaciones permitidas que se mantengan por debajo de los límites máximos de un plan. No obstante, si necesita un enfoque de aprovisionamiento basado en script o en código, [Azure PowerShell](search-manage-powershell.md) o la [API REST de administración](/rest/api/searchmanagement/services) son soluciones alternativas.

## <a name="terminology-replicas-and-partitions"></a>Terminología: réplicas y particiones

|||
|-|-|
|*Particiones* | Proporciona almacenamiento de índices y E/S para realizar operaciones de lectura y escritura (por ejemplo, volver a generar o actualizar un índice). Cada partición tiene una parte del índice total. Si asigna tres particiones, el índice se divide en tercios. |
|*Réplicas* | Instancias del servicio de búsqueda, que se utilizan principalmente para equilibrar la carga de las operaciones de consulta. Cada réplica es una copia de un índice. Si asigna tres réplicas, tendrá tres copias de un índice disponibles para atender las solicitudes de consulta.|

## <a name="when-to-add-nodes"></a>Cuándo agregar nodos

Inicialmente, se asigna un servicio a un nivel mínimo de recursos que consta de una partición y una réplica. 

Un único servicio debe tener recursos suficientes para controlar todas las cargas de trabajo (indexación y consultas). Ninguna carga de trabajo se ejecuta en segundo plano. Puede programar la indexación en horas en las que las solicitudes de consulta son menos frecuentes por naturaleza, pero el servicio no dará prioridad a una tarea sobre otra. Además, una determinada cantidad de redundancia suaviza el rendimiento de la consulta cuando los servicios o nodos se están actualizando internamente.

Como norma general, las aplicaciones de búsqueda tienden a necesitar más réplicas que particiones, sobre todo cuando las operaciones de servicio están orientadas a las cargas de trabajo de consulta. En la siguiente sección sobre [alta disponibilidad](#HA) se explica el motivo.

> [!NOTE]
> La adición de más réplicas o particiones aumenta el costo de ejecución del servicio y puede generar pequeñas variaciones en cómo se ordenan los resultados. Asegúrese de activar la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para comprender las implicaciones que tiene en la facturación el agregar más nodos. El [gráfico siguiente](#chart) puede ayudarle a establecer una referencia cruzada con el número de unidades de búsqueda necesarias para una configuración específica. Para obtener más información sobre cómo las réplicas adicionales afectan al procesamiento de las consultas, visite [Organización de los resultados](search-pagination-page-layout.md#ordering-results).

## <a name="how-to-allocate-replicas-and-partitions"></a>Cómo asignar réplicas y particiones

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y seleccione su servicio de búsqueda.

1. En **Configuración**, abra la página **Escala** para modificar réplicas y particiones. 

   En la siguiente captura de pantalla se muestra un servicio estándar aprovisionado con una réplica y una partición. La fórmula de la parte inferior indica cuántas unidades de búsqueda se usan (1). Si el precio por unidad era de 100 USD (no un precio real), el costo de ejecución de este servicio sería, de media, de 100 USD.

   ![Página de escala que muestra los valores actuales](media/search-capacity-planning/1-initial-values.png "Página de escala que muestra los valores actuales")

1. Use el control deslizante para aumentar o reducir el número de particiones. La fórmula de la parte inferior indica cuántas unidades de búsqueda se usan.

   En este ejemplo se duplica la capacidad, con dos réplicas y particiones de forma individual. Observe el recuento de unidades de búsqueda; ahora es cuatro porque la fórmula de facturación son las réplicas multiplicadas por las particiones (2 x 2). Cuanto más se duplica la capacidad, más se duplica el costo de ejecución del servicio. Si el costo de la unidad de búsqueda era de 100 USD, la nueva factura mensual sería ahora de 400 USD.

   Para ver los costos por unidad actuales de cada nivel, visite la [Página de precios](https://azure.microsoft.com/pricing/details/search/).

   ![Adición de réplicas y particiones](media/search-capacity-planning/2-add-2-each.png "Adición de réplicas y particiones")

1. Haga clic en **Guardar** para confirmar los cambios.

   ![Confirmación de cambios en la escala y facturación](media/search-capacity-planning/3-save-confirm.png "Confirmación de cambios en la escala y facturación")

   Los cambios en la capacidad tardan varias horas en completarse. No se puede cancelar una vez que se ha iniciado el proceso y no hay ningún tipo de supervisión en tiempo real de los ajustes de réplica y partición. Sin embargo, el siguiente mensaje se puede seguir viendo mientras se están realizando los cambios.

   ![Mensaje de estado del portal](media/search-capacity-planning/4-updating.png "Mensaje de estado del portal")

> [!NOTE]
> Una vez que se aprovisiona un servicio, no se puede actualizar a un plan superior. Debe crear un servicio de búsqueda en el nivel nuevo y volver a cargar los índices. Consulte [Creación de un servicio Azure Cognitive Search mediante Azure Portal](search-create-service-portal.md) para obtener ayuda con el proceso de aprovisionamiento de servicios.
>
> Además, las particiones y las réplicas las administra el servicio de manera exclusiva e interna. No hay concepto de afinidad de procesador ni de asignación de una carga de trabajo a un nodo específico.
>

<a id="chart"></a>

## <a name="partition-and-replica-combinations"></a>Combinaciones de particiones y réplicas

Un servicio del nivel Básico puede tener exactamente 1 partición y hasta 3 réplicas para un límite máximo de 3 SU. El único recurso que puede ajustarse son las réplicas. Se necesita un mínimo de 2 réplicas para lograr una alta disponibilidad en las consultas.

Todos los servicios de búsqueda de los niveles Estándar y Almacenamiento optimizado pueden suponer las siguientes combinaciones de réplicas y particiones, con el límite de 36 unidades de búsqueda (SU). 

|   | **1 partición** | **2 particiones** | **3 particiones** | **4 particiones** | **6 particiones** | **12 particiones** |
| --- | --- | --- | --- | --- | --- | --- |
| **1 réplica** |1 unidad de búsqueda |2 unidades de búsqueda |3 unidades de búsqueda |4 unidades de búsqueda |6 unidades de búsqueda |12 unidades de búsqueda |
| **2 réplicas** |2 unidades de búsqueda |4 unidades de búsqueda |6 unidades de búsqueda |8 unidades de búsqueda |12 unidades de búsqueda |24 unidades de búsqueda |
| **3 réplicas** |3 unidades de búsqueda |6 unidades de búsqueda |9 unidades de búsqueda |12 unidades de búsqueda |18 unidades de búsqueda |36 unidades de búsqueda |
| **4 réplicas** |4 unidades de búsqueda |8 unidades de búsqueda |12 unidades de búsqueda |16 unidades de búsqueda |24 unidades de búsqueda |N/D |
| **5 réplicas** |5 unidades de búsqueda |10 unidades de búsqueda |15 unidades de búsqueda |20 unidades de búsqueda |30 unidades de búsqueda |N/D |
| **6 réplicas** |6 unidades de búsqueda |12 unidades de búsqueda |18 unidades de búsqueda |24 unidades de búsqueda |36 unidades de búsqueda |N/D |
| **12 réplicas** |12 unidades de búsqueda |24 unidades de búsqueda |36 unidades de búsqueda |N/D |N/D |N/D |

En el sitio web de Azure se explican con detalle la capacidad, los precios y las unidades de búsqueda. Para obtener más información, consulte [Detalles de precios](https://azure.microsoft.com/pricing/details/search/).

> [!NOTE]
> El número de réplicas y particiones se dividirse equitativamente en 12 (en concreto, 1, 2, 3, 4, 6, 12). Esto se debe a que Azure Cognitive Search divide previamente cada índice en 12 particiones para que se pueda repartir en porciones iguales entre todas las particiones. Por ejemplo, si su servicio tiene tres particiones y crea un nuevo índice, cada partición contendrá 4 particiones del índice. La manera en que Azure Cognitive Search particiona un índice es un detalle de implementación, sujeto a cambios en la futura versión. Aunque el número es 12 hoy, no debe esperar que ese número se siempre 12 en el futuro.
>

<a id="HA"></a>

## <a name="high-availability"></a>Alta disponibilidad

Dado que es sencillo y relativamente rápido escalar verticalmente, generalmente se recomienda que comience con una partición y una o dos réplicas, y que después escale verticalmente conforme se crean volúmenes de consulta. Las cargas de trabajo de consulta se ejecutan principalmente en réplicas. Es probable que necesite más réplicas si requiere más rendimiento o alta disponibilidad.

Las recomendaciones generales para alta disponibilidad son:

* Dos réplicas para alta disponibilidad de cargas de trabajo de solo lectura (consultas)

* Tres o más réplicas para lograr una alta disponibilidad en las cargas de trabajo de lectura y escritura (se agregan, actualizan o eliminan consultas e indexación como documentos individuales).

Los Acuerdos de Nivel de Servicio (SLA) de Azure Cognitive Search están destinados a las operaciones de consulta y actualizaciones de índices que constan de procesos de incorporación, actualización o eliminación de documentos.

El nivel básico alcanza el límite en una partición y tres réplicas. Si desea la flexibilidad de responder inmediatamente a las fluctuaciones en la demanda para el rendimiento de indexación y consulta, considere uno de los niveles Estándar.  Si descubre que el crecimiento de sus requisitos de almacenamiento es mucho más rápido que el del rendimiento de consultas, tenga en cuenta uno de los niveles Almacenamiento optimizado.

## <a name="disaster-recovery"></a>Recuperación ante desastres

En la actualidad no hay ningún mecanismo integrado para la recuperación ante desastres. La adición de particiones o réplicas sería la estrategia equivocada para cumplir los objetivos de recuperación ante desastres. El enfoque más común es agregar redundancia en el nivel de servicio mediante la configuración de un segundo servicio de búsqueda en otra región. Al igual que con la disponibilidad durante la regeneración de índices, la lógica de conmutación por error o redireccionamiento debe proporcionarse en el código.

## <a name="estimate-replicas"></a>Estimación de réplicas

En un servicio de producción, debe asignar tres réplicas para cumplir el Acuerdo de Nivel de Servicio. Si experimenta un rendimiento lento de consultas, puede agregar réplicas para que se pongan en línea copias adicionales del índice para admitir mayores cargas de trabajo de consultas y equilibrar la carga de las solicitudes por las diversas réplicas.

No se proporcionan instrucciones sobre cuántas réplicas se necesitan para acomodar las cargas de consulta. El rendimiento de consulta depende de la complejidad de la consulta y de las cargas de trabajo competitivas. Si bien la adición de réplicas genera claramente un mejor rendimiento, el resultado final no será estrictamente lineal: la adición de tres réplicas no garantiza el triple rendimiento.

Para obtener una guía sobre cómo estimar las QPS de la solución, consulte [Escalado para mejorar el rendimiento](search-performance-optimization.md) y [Supervisión de consultas](search-monitor-queries.md)

## <a name="estimate-partitions"></a>Estimación de particiones

El [plan que elija](search-sku-tier.md) determina el tamaño y la velocidad de la partición y cada plan se optimiza alrededor de un conjunto de características que se adapta a varios escenarios. Si elige un plan de gama superior, puede que necesite menos particiones que si elige S1. Una de las preguntas que tendrá que responder mediante pruebas autodirigidas es si una partición más grande y más cara aumentará más el rendimiento que dos particiones más baratas en un servicio aprovisionado con un plan inferior.

Las aplicaciones de búsqueda que requieren una actualización de datos casi en tiempo real necesitan una proporción mayor de particiones que de réplicas. Al agregarse particiones, se distribuyen las operaciones de lectura y escritura entre un número mayor de recursos de proceso. Además, se cuenta con más espacio en disco para almacenar documentos e índices adicionales.

Las consultas en índices de mayor tamaño tardan más tiempo en realizarse. Por lo tanto, es posible que con cada aumento incremental de las particiones sea necesario también un aumento menor, pero proporcional, de las réplicas. La complejidad y el volumen de las consultas afectarán a la rapidez con que se ejecuta la consulta.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Selección de un plan de tarifa de Azure Cognitive Search](search-sku-tier.md)