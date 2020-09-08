---
title: 'Información de Marketplace: Marketplace comercial de Microsoft, Microsoft AppSource y Azure Marketplace'
description: Acceda a un resumen de análisis web de marketplace, que permite medir la involucración del cliente en Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 07/22/2019
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: cc950c4915a57dd949941fbb9802c51a47c9a00d
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88607141"
---
# <a name="marketplace-insights-dashboard-in-partner-center"></a>Panel de Información de Marketplace en Centro de partners

En este artículo se proporciona información sobre el panel Información de Marketplace del Centro de partners. En este panel se muestra un resumen de los análisis web de marketplace que permite a los editores medir la participación del cliente en sus respectivas páginas de detalles de productos en las tiendas en línea de marketplace comercial: Microsoft AppSource y Azure Marketplace.

## <a name="marketplace-insights-dashboard"></a>Panel de información de Marketplace

Para acceder al **panel Información de Marketplace** en el Centro de partners, abra la **[pestaña Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)** en Marketplace comercial.

Puede ver representaciones gráficas de los elementos siguientes:  

- [Resumen de Información de Marketplace](#marketplace-insights-summary)
- [Visitas a la página según la geografía](#page-visits-by-geography)  
- [Tendencia de visitas a la página frente a visitantes únicos](#page-visits-versus-unique-visitors-trend)
- [Llamada a la acción (CTA) frente a visitantes únicos con llamadas a la acción](#call-to-action-versus-unique-visitors-with-ctas)
- [Visitas a la página y llamadas a la acción según las ofertas](#page-visits-and-calls-to-action-by-offers)
- [Tendencia del porcentaje de las llamadas a la acción](#call-to-action-percentage-trend)
- [Visitas a la página y llamadas a la acción según los dominios de referencia](#page-visits-and-calls-to-action-by-referral-domains)
- [Tabla de detalles de Información de Marketplace](#marketplace-insights-details-table)

La latencia máxima entre los usuarios que visitan ofertas en Azure Marketplace o AppSource y los que presentan informes en el Centro de partners es de 48 horas.

>[!NOTE]
> Para obtener definiciones detalladas de la terminología de análisis, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./faq-terminology.md).

### <a name="insights-dashboard-layout"></a>Diseño del panel de información

Vea las métricas de marketplace comercial de varias maneras:

- Pestañas de tienda en línea
- Filtros de página
- Filtros de fecha

**Pestañas de tienda en línea**: puede ver las métricas de las ofertas por separado mediante las pestañas AppSource y Azure Marketplace. Seleccione las ofertas en la lista desplegable de ofertas del lado derecho para obtener una visualización de las métricas de las ofertas seleccionadas. De forma predeterminada, se seleccionan todas las opciones.

![Lista desplegable de las ofertas del panel Información del Centro de partners](./media/insights-offer-dropdown.png)

**Filtros de la página Información**: estos filtros se aplican a nivel de la oferta (página de detalles del producto). Puede seleccionar varios filtros para los criterios que quiere ver. Este filtro se aplica a toda la sección Información de Marketplace, incluidos los gráficos y los detalles.

![Filtros de la página del panel Información del Centro de partners](./media/insights-page-filter.png)

- Los nombres de las ofertas solo se muestran para las ofertas que tienen visitas a la página en el intervalo de fechas seleccionado.  
- La selección predeterminada es "Todo" para cada una de las opciones de filtro.
- Los filtros aplicados muestran el número de selecciones realizadas. Los filtros aplicados no se mostrarán para la selección predeterminada "Todo".

![Filtros de Información del Centro de partners aplicados](./media/insights-page-filter-two.png)

**Filtros de datos de Información**: este filtro se aplica a toda la sección Información de Marketplace. Los filtros pueden incluir intervalos de fechas predeterminados o un intervalo de fechas personalizado.

![Filtros de fecha de Información del Centro de partners](./media/insights-date-range.png)

## <a name="marketplace-insights-summary"></a>Resumen de Información de Marketplace

En la sección Resumen de Información de Marketplace se muestra un recuento de las **visitas a la página**, las **llamadas a la acción** y los **visitantes únicos** para el intervalo de fechas seleccionado.

### <a name="page-visits"></a>Visitas a la página

Este número representa el número de sesiones de usuario distintas en la página de ofertas (página de detalles del producto) para un intervalo de fechas seleccionado. El indicador de porcentaje rojo o verde representa el porcentaje de crecimiento de las visitas a la página. El gráfico de tendencias representa el recuento mensual de visitas a la página.

### <a name="unique-visitors"></a>Visitantes únicos

Este número representa el recuento de visitantes distinto durante el intervalo de fechas seleccionado para las ofertas seleccionadas en el filtro de la página. Un visitante que haya visitado una o varias páginas de detalles de productos se contará como un visitante único.

### <a name="call-to-action"></a>Llamada a la acción

Este número representa el recuento de clics en el botón **Llamada a la acción** completados en la página de ofertas (página de detalles del producto). Las **llamadas a la acción** se cuentan cuando los usuarios seleccionan los botones **Obtener ahora**, **Prueba gratuita**, **Contacto** o **Versión de prueba**.

![Resumen de la llamada a la acción de Información del Centro de partners](./media/insights-summary.png)

## <a name="page-visits-by-geography"></a>Visitas a la página según la geografía

En el mapa térmico siguiente se muestra el recuento de **visitas a la página**, **llamadas a la acción** y **visitantes únicos según el país del cliente**. Los valores más altos de las visitas a la página se representan mediante colores de mapa más oscuros y los más bajos se representan mediante colores de mapa más claros.

![Propagación geográfica de Información del Centro de partners](./media/insights-geography.png)

El mapa térmico incluye las siguientes funcionalidades:

- El mapa térmico cuenta con una cuadrícula complementaria para ver los detalles de las **visitas a la página**, las **llamadas a la acción** y los **visitantes únicos** de una ubicación específica. Si lo prefiere, puede hacer zoom en una ubicación específica.  
- La **dispersión en países o regiones** es el recuento de países o regiones desde los que los clientes han comunicado visitas a la página durante el rango de fechas seleccionado.
- Puede buscar y seleccionar un país en la cuadrícula para ampliar la ubicación en el mapa. Para revertir la vista original, seleccione **Inicio** en el mapa.

## <a name="page-visits-versus-unique-visitors-trend"></a>Tendencia de visitas a la página frente a visitantes únicos

En las columnas siguientes se representa el recuento de visitas a la página mensuales, que se muestran en el eje Y (eje del lado izquierdo del gráfico). La línea de tendencia representa la tendencia mensual de visitantes únicos, que se muestra en el eje Y secundario (eje del lado derecho del gráfico), de las ofertas publicadas en las tiendas en línea: Azure Marketplace y AppSource.

![Tendencia de visitas a la página de Información del Centro de partners frente a visitantes únicos](./media/insights-page-vists-unique-visitors.png)

## <a name="call-to-action-versus-unique-visitors-with-ctas"></a>Llamada a la acción frente a visitantes únicos con llamadas a la acción

Las columnas apiladas representan las llamadas a la acción mensuales, que se dividen en tipos de CTA (**Obtener ahora**, **Contacto**y **Prueba gratuita**) y se trazan en el eje Y (eje del lado izquierdo de la página). La línea de tendencia representa la tendencia mensual de visitantes únicos con llamadas a la acción, que se muestra en el eje Y secundario (eje del lado derecho del gráfico), de las ofertas publicadas en Azure Marketplace y AppSource.

![Llamada a la acción de Información del Centro de partners frente a visitantes únicos con llamadas a la acción](./media/insights-call-to-action-unique-visitors.png)

## <a name="page-visits-and-calls-to-action-by-offers"></a>Visitas a la página y llamadas a la acción según las ofertas

El gráfico circular exterior representa el desglose de las **visitas a la página** en función de las ofertas que haya publicado en Marketplace y seleccionado en el filtro. El gráfico interno representa el desglose de las **llamadas a la acción** de las mismas ofertas.

![Visitas a la página de Información del Centro de partners y llamadas a la acción según las ofertas](./media/insights-page-visits-and-cta-by-offer.png)

## <a name="call-to-action-percentage-trend"></a>Tendencia del porcentaje de las llamadas a la acción

La **tendencia del porcentaje de las llamadas a la acción** presenta el porcentaje de las llamadas a la acción de las ofertas publicadas en Marketplace. Porcentaje de llamadas a la acción = (llamadas a la acción/visitas a la página) * 100.

![Tendencia del porcentaje de llamadas a la acción de Información del Centro de partners](./media/insights-call-to-action-percentage-trend.png)

## <a name="page-visits-and-calls-to-action-by-referral-domains"></a>Visitas a la página y llamadas a la acción según los dominios de referencia

En el gráfico siguiente se muestran los 50 dominios de referencia principales. Al seleccionar un dominio de referencia específico, se muestra la tendencia mensual de las visitas a la página y las llamadas a la acción en el gráfico de la derecha.

![Visitas a la página de Información del Centro de partners y llamadas a la acción según las campañas y los dominios de referencia](./media/insights-page-visits-call-to-actions.png)

## <a name="marketplace-insights-details-table"></a>Tabla de detalles de Información de Marketplace

En esta tabla se proporciona una vista de lista de las visitas a la página y las llamadas a la acción de las ofertas seleccionadas ordenadas por fecha.

![Tabla de detalles de Información del Centro de partners](./media/insights-details-page.png)

- Los datos se pueden extraer en un archivo TSV si el recuento de los registros es inferior a 1000.
- Si el número de registros es superior a 1000, los datos exportados se colocarán de forma asincrónica en una página de descargas durante los próximos 30 días.
- Filtre los datos por los nombres de las ofertas y las campañas para mostrar los datos que le interesan.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los informes de análisis disponibles en el marketplace comercial del Centro de partners, consulte [Análisis del marketplace comercial en el Centro de partners](./analytics.md).
- Para ver los gráficos, las tendencias y los valores de datos agregados que resuman la actividad de la oferta en el marketplace, consulte [Panel Resumen de los análisis de marketplace comercial](./summary-dashboard.md).
- Para información sobre los pedidos en un formato gráfico que se pueda descargar, consulte [Panel Pedidos de los análisis de marketplace comercial](./orders-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](./usage-dashboard.md).
- Para información detallada sobre los clientes, incluidas las tendencias de crecimiento, consulte [Panel Cliente de los análisis de marketplace comercial](./customer-dashboard.md).
- Para ver una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](./downloads-dashboard.md).
- Para tener una vista consolidada de comentarios de los clientes sobre las ofertas de Azure Marketplace y AppSource, consulte [Panel Calificaciones y opiniones de los análisis de marketplace comercial](./ratings-reviews.md).
- Para ver las preguntas más frecuentes sobre los análisis de marketplace comercial y un diccionario completo de términos de datos, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./faq-terminology.md).
