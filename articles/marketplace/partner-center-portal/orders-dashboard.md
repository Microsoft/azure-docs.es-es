---
title: Panel Pedidos del Centro de partners de los análisis del marketplace comercial, Microsoft AppSource y Azure Marketplace
description: Obtenga información sobre cómo acceder a informes analíticos sobre los pedidos de ofertas del marketplace comercial en un formato gráfico y descargable.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 07/22/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: d5adc1bfe19de48568d0e77bb488bea0e5a02818
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327385"
---
# <a name="orders-dashboard-in-commercial-marketplace-analytics"></a>Panel Pedidos de los análisis de marketplace comercial

En este artículo se proporciona información sobre el **panel Pedidos** del Centro de partners. En este panel se muestra información sobre pedidos en formato gráfico y descargable.

Para obtener acceso al **panel Pedidos** de las herramientas de análisis del Centro de partners, abra el **[panel Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)** bajo Marketplace comercial.

>[!NOTE]
> Para obtener definiciones detalladas de la terminología de análisis, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./faq-terminology.md).

## <a name="orders-dashboard"></a>Panel Pedidos

En el **panel Pedidos** del menú **Analizar** se muestran los pedidos actuales para todas las ofertas de SaaS. Puede ver representaciones gráficas de los elementos siguientes:

- [Resumen del pedido](#order-summary)
- [Pedidos por geografía](#orders-by-geography)
- [Pedidos por ofertas](#orders-by-offers)
- [Tendencia de pedidos por sitio y puesto](#orders-trend-per-site-versus-per-seat)
- [Pedidos por planes](#orders-by-plans)
- [Tendencia de pedidos y puestos](#orders-and-seats-trend)
- [Tabla de detalles de pedidos](#order-details-table)

La latencia máxima entre la creación de pedidos y la generación de informes en el Centro de partners es de 48 horas.

## <a name="order-dashboard-details"></a>Detalles del panel Pedidos

En esta sección se describen los informes analíticos con más detalle.

### <a name="order-summary"></a>Resumen del pedido

En la sección Resumen del pedido se muestra un recuento de todos los pedidos comprados (excepto los cancelados), los pedidos cancelados y los puestos.

El valor de porcentaje junto al de Total de pedidos representa el crecimiento en comparación con el intervalo de fechas anterior.

![Resumen de pedidos de Analizar del Centro de partners](./media/order-summary.png)

- Un triángulo verde que apunta hacia arriba indica una tendencia de crecimiento positivo.
- Un triángulo rojo que apunta hacia abajo indica una tendencia de crecimiento negativo en relación con el mes anterior.
- Las tendencias de crecimiento están representadas por micrográficos de barras. Para mostrar el valor de cada mes, puede mantener el puntero sobre las columnas del gráfico.
- Los pedidos cancelados son el número de pedidos que se compraron previamente y, luego, se cancelaron durante el intervalo de fechas seleccionado.
- Los puestos hacen referencia a un recuento de los puestos creados durante el intervalo de fechas seleccionado.

### <a name="orders-by-geography"></a>Pedidos por geografía

En el mapa térmico **Pedidos por geografía** se muestra el recuento de los pedidos en un mapa del mundo y los puestos asignados según el país o la región del cliente. Este mapa térmico funciona igual que el **[mapa térmico Clientes por geografía](./customer-dashboard.md#customer-by-geography)** .

![Pedidos por geografía de Analizar del Centro de partners](./media/orders-by-geography.png)

### <a name="orders-by-offers"></a>Pedidos por ofertas

El gráfico de anillos **Pedidos por ofertas** organiza los pedidos (incluidos los cancelados) según sus nombres de oferta.

- Las ofertas principales se muestran en el gráfico y el resto se agrupan como "Todo el resto".
- Puede seleccionar las ofertas específicas de la leyenda para mostrar solo esas ofertas en el gráfico.
- Al mantener el puntero sobre un sector del gráfico aparece el número de pedidos y el porcentaje de esa oferta en comparación con el número total de pedidos de todas las demás ofertas.
- La **tendencia de los pedidos por ofertas** muestra las tendencias de crecimiento mensualmente. La columna de mes representa el número de pedidos por nombre de la oferta. El gráfico de líneas muestra la tendencia del porcentaje de crecimiento trazada en un eje z.
- Puede usar el control deslizante de la parte superior del gráfico para desplazarse hacia la derecha y la izquierda a lo largo del eje x y centrarse en puntos de datos específicos.
- Puede mostrar el gráfico de tendencias seleccionando un elemento específico en la leyenda.
- También puede mostrar las tendencias y los datos de los **pedidos cancelados**. El gráfico funcionará de la misma manera que el de **Pedidos por ofertas**.

### <a name="orders-trend-per-site-versus-per-seat"></a>Tendencia de pedidos por sitio y puesto

El gráfico de anillos **por sitio y puesto** representa el desglose de los pedidos SaaS por sitio y SaaS por puesto adquiridos por los clientes (este gráfico incluye pedidos cancelados). El gráfico de columnas representa la tendencia de los pedidos SaaS por sitio y por puesto que compran los clientes (este gráfico incluye pedidos cancelados).

### <a name="orders-by-plans"></a>Pedidos por planes

En el gráfico **Pedidos por planes** se representa la tendencia de los pedidos a nivel de plan de todas las ofertas (esto incluye los pedidos cancelados). En el gráfico de anillos se representa el desglose de los cinco primeros pedidos de planes y en el gráfico de columnas se ilustra la tendencia de los pedidos de los primeros cinco planes.

### <a name="orders-and-seats-trend"></a>Tendencia de pedidos y puestos

El gráfico **Tendencia de pedidos y puestos** presenta las 50 ofertas principales con el mayor número de pedidos. Se muestran en una tabla de clasificación y se ordenan por el mayor número de pedidos y porcentaje de pedidos.

- **Pedidos por planes**: seleccione una oferta para ver el desglose del recuento de pedidos de los cinco planes principales del gráfico.
- **Puestos pos planes**: tendencia mensual de puestos de los cinco planes principales. Si la oferta que selecciona no es una oferta por puesto, no se mostrará ningún dato aquí.

### <a name="canceled-orders-by-offers"></a>Pedidos cancelados por ofertas

En el gráfico de anillos **Pedidos cancelados por ofertas** se organizan todos los pedidos cancelados según sus nombres de oferta. Las ofertas principales se muestran en el gráfico y el resto se agrupan como "Todo el resto". Puede seleccionar las ofertas específicas de la leyenda para mostrarlas en el gráfico.

- Al mantener el puntero sobre un sector del gráfico, se muestra el número de pedidos y el porcentaje de la oferta seleccionada en comparación con el número total de pedidos de todas las demás ofertas.
- En el gráfico de columnas se muestran las tendencias mensualmente. Las columnas representan el número de pedidos cancelados por nombre de oferta. Puede usar el control deslizante de la parte superior del gráfico para desplazarse hacia la derecha y la izquierda a lo largo del eje x y centrarse en puntos de datos específicos. Puede mostrar el gráfico de tendencias seleccionando un elemento específico en la leyenda.

### <a name="order-details-table"></a>Tabla de detalles de pedidos

En la tabla de detalles de pedidos se muestra una lista numerada de los 1000 pedidos principales ordenados por fecha de adquisición.

- Cada columna de la cuadrícula se puede ordenar.
- Los datos se pueden extraer en un archivo TSV si el recuento de los registros es inferior a 1000.
- Si el número de registros es superior a 1000, los datos exportados se colocarán de forma asincrónica en una página de descargas durante los próximos 30 días.
- Aplique filtros a la **tabla de detalles de pedidos** a fin de mostrar solo los datos que le interesen. Filtre por país o región, tipo de licencia de Azure, tipo de licencia del marketplace comercial, tipo de oferta, estado de pedido, evaluaciones gratuitas, identificador de suscripción del marketplace comercial, identificador de cliente y nombre de la empresa.
- Dado que las ofertas de SaaS compradas a través de Azure Marketplace o AppSource no requieren una suscripción a Azure, el identificador de suscripción de Marketplace aparecerá como 00000000-0000-0000-0000-000000000000 en la sección **Datos de pedidos detallados**.

#### <a name="orders-page-filters"></a>Filtros de la página de pedidos

Estos filtros se aplican a nivel de página.

Puede seleccionar varios filtros para representar el gráfico para los criterios que quiera ver, así como los datos que quiera que se muestren en la cuadrícula o exportación de **datos detallados del pedido**. Los filtros se aplican a los datos extraídos del intervalo de datos que ha seleccionado en la esquina superior derecha de la página de pedidos.

- Los tipos de oferta y los nombres de ofertas solo se muestran para las ofertas para las que ha realizado pedidos durante el intervalo de fechas seleccionado. Los nombres de oferta de la lista se muestran para los tipos de ofertas que se seleccionan en la lista.
- Los filtros aplicados muestran las métricas totales de cada selección para cada filtro seleccionado. Los filtros aplicados no se muestran cuando se elige la selección predeterminada.
- Si se selecciona la opción **Todos** para una de las listas desplegables, se agregarán todas las métricas de la página seleccionada. Por ejemplo: "Todos" en la opción de filtro de tipos de ofertas significa que se han seleccionado todos los tipos de ofertas. Esta es la selección predeterminada para las listas desplegables. En las visualizaciones de filtros aplicados no se mostrará nada cuando se seleccione la opción **Todos**.
- **Selección de varios valores**: todas las métricas de la página se agregarán para todas las selecciones realizadas en la lista desplegable. Si se realizan selecciones múltiples, el filtro aplicado mostrará el recuento de todas las selecciones realizadas. Vea la imagen siguiente a modo de referencia.

    ![Pedido de Analizar del Centro de partners con varios valores aplicados al filtro](./media/filters-applied.png)

- **Selección de un solo valor**: si se selecciona un valor, el filtro aplicado mostrará el recuento del filtro seleccionado. Vea la imagen siguiente como referencia.

     ![Pedido de Analizar del Centro de partners con un solo valor aplicado al filtro](./media/filters-applied-single.png)

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los informes de análisis disponibles en el marketplace comercial del Centro de partners, consulte [Análisis del marketplace comercial en el Centro de partners](./analytics.md).
- Para ver los gráficos, las tendencias y los valores de datos agregados que resumen la actividad de la oferta en el marketplace, consulte [Panel Resumen de los análisis de marketplace comercial](./summary-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](./usage-dashboard.md).
- Para obtener información detallada acerca de los clientes, incluidas las tendencias de crecimiento, consulte [Panel Cliente de los análisis de marketplace comercial](./customer-dashboard.md).
- Para obtener una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](./downloads-dashboard.md).
- Para tener una vista consolidada de comentarios de los clientes sobre las ofertas de Azure Marketplace y AppSource, consulte [Panel Calificaciones y opiniones de los análisis de marketplace comercial](./ratings-reviews.md).
- Para ver las preguntas más frecuentes sobre los análisis de marketplace comercial y un diccionario completo de términos de datos, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./faq-terminology.md).
