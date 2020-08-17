---
title: Panel Resumen para los análisis del Centro de partners en Marketplace comercial
description: Aprenda a acceder a gráficos, tendencias y valores de datos agregados que resumen la actividad de Marketplace desde el panel Resumen en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 12/11/2019
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: 8e58de7975bc7f5de1fe2ad71f97c02c901c0c21
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87304129"
---
# <a name="summary-dashboard-in-commercial-marketplace-analytics"></a>Panel Resumen de los análisis de marketplace comercial

En este artículo se proporciona información sobre el panel Resumen del Centro de partners. Este panel muestra gráficos, tendencias y valores de datos agregados que resumen la actividad de Marketplace para las ofertas.

Para acceder al panel Resumen, abra el **[panel Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)** en Marketplace comercial.

>[!NOTE]
> Para obtener definiciones detalladas de la terminología de análisis, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./faq-terminology.md).

## <a name="summary-dashboard"></a>Panel Resumen

El panel **Resumen** presenta información general en función de cada tipo de oferta. **Información** muestra información crítica de un vistazo y proporciona una visión amplia de la actividad de ventas de sus ofertas. Puede ver estos informes mediante el panel **Resumen**. En este artículo se ofrece información más detallada sobre cada uno de los siguientes elementos:

- [Intervalo de fechas](#date-range)
- [Sección Resumen](#summary-section)
- [Clientes por geografía](#customers-by-geography)
- [Gráficos de tendencias de crecimiento](#growth-trend-charts)
- [Tabla de clasificación de clientes](#customer-leaderboard)
- [Tendencia de número de puestos](#seat-count-trend)
- [Tendencia de los pedidos de SaaS de evaluación gratuita](#free-trials-saas-orders-trend)

## <a name="elements-of-the-summary-dashboard"></a>Elementos del panel Resumen

En las secciones siguientes se describe cómo usar el panel Resumen y cómo leer los datos.

### <a name="date-range"></a>Intervalo de fechas

Puede encontrar una selección de intervalo de fecha en la esquina superior derecha de cada página. Puede personalizar salida de los gráficos de la página **Resumen** seleccionando un intervalo de fechas basado en los últimos 3, 6 o 12 meses, o seleccionando un intervalo de fechas personalizado con una duración máxima de 12 meses. El intervalo de fechas predeterminado es de seis meses.

![Panel Analizar del Centro de partners](./media/analyze-dashboard.png)

### <a name="summary-section"></a>Sección Resumen

La sección Resumen muestra un recuento de todos los pedidos creados, clientes adquiridos y uso notificado durante el intervalo de fechas seleccionado. La parte correspondiente al mes en curso se excluirá del cálculo de estas métricas. Por ejemplo: Si ha seleccionado el intervalo de tiempo de 6 meses, se calculan las horas de uso de los seis meses anteriores al mes actual. Si se selecciona un intervalo de fechas personalizado, se excluirá del cálculo una cantidad parcial del mes en curso.

![Tendencias de crecimiento en el panel Resumen](./media/summary-summary-section.png)

#### <a name="reading-the-summary-section"></a>Lectura de la sección Resumen

- **Pedidos**: Recuento de todos los pedidos adquiridos (excluye los pedidos cancelados) de las ofertas que ha publicado hasta el momento.
- **Clientes**: Recuento de todos los clientes que compraron sus ofertas y que tienen al menos un pedido no cancelado.
- **Horas de uso normalizado**: Definidas como las horas de uso normalizado que justifican el número de núcleos de máquina virtual ([número de núcleos de máquina virtual] x [horas de uso sin procesar]). Las máquinas virtuales designadas como "SHAREDCORE" usan 1/6 (o 0,1666) como multiplicador de [número de núcleos de máquina virtual].
- **Horas de uso sin procesar**: La cantidad de tiempo que las máquinas virtuales se han ejecutado en términos de horas. El valor de porcentaje situado junto a **total de pedidos**, **clientes totales**, **horas de uso normalizado**, **horas de uso sin procesar**, **visitas de página** y **llamada a acciones** representa la cantidad de crecimiento de uso para el intervalo de fechas seleccionado ([uso del último mes-uso del primer mes])/uso del primer mes). Tal y como se ha dicho anteriormente, se excluirá de esta métrica una cantidad parcial del mes en curso.
- **Tendencias de crecimiento**: Si mantiene el mouse sobre las columnas del gráfico, los gráficos de barras muestran el valor de cada mes.
- **Triángulo verde apuntando hacia arriba**: Indica una tendencia de crecimiento positiva.
- **Triángulo rojo apuntando hacia abajo**: Indica una tendencia de crecimiento negativo en relación con el mes anterior.

### <a name="customers-by-geography"></a>Clientes por geografía

El mapa térmico **Clientes por geografía** muestra un recuento de los clientes en un mapa del mundo.

![Clientes por geografía en el panel Resumen](./media/summary-customers-by-geography.png)

- Puede mover el mapa para ver la ubicación exacta.
- Puede hacer zoom en una ubicación específica.
- El mapa térmico tiene una cuadrícula complementaria para ver los detalles del recuento de clientes, recuento de pedidos y horas de uso normalizado en la ubicación específica.
- Puede buscar y seleccionar un país o región en la cuadrícula para ampliar la ubicación en el mapa. Revierta a la vista original presionando el botón **Inicio** en el mapa.
- Un **nuevo** cliente ha comprado una de sus ofertas por primera vez durante el mes dentro del intervalo de fechas seleccionado.

### <a name="growth-trend-charts"></a>Gráficos de tendencias de crecimiento

Puede ver las tendencias en función del crecimiento de los **pedidos adquiridos** (incluye pedidos cancelados), **clientes adquiridos** (incluye a los clientes perdidos) y el **uso** notificado, que se muestran mensualmente de acuerdo con el intervalo de fechas seleccionado. Puede analizar más a fondo estas tendencias seleccionando los vínculos debajo del gráfico, que llevan a las respectivas páginas de **pedido**, **uso**, **cliente** o **Información de Marketplace**.

La oferta de Marketplace de gráficos **Tendencia de las visitas a la página y las llamadas a la acción** se muestra para Azure Marketplace y AppSource en dos pestañas.

![Gráficos Tendencia de las visitas a la página y las llamadas a la acción en el panel Resumen](./media/summary-page-visits-and-cta.png)

El gráfico de **pedidos por ofertas** organiza los pedidos de acuerdo al nombre de la oferta.

El gráfico circular de **pedidos por canal de ventas** organiza los pedidos (incluidos los pedidos que los clientes han cancelado) durante el intervalo de fechas seleccionado, por canal de ventas. Un canal de ventas es el tipo de contrato de licencia que usan los clientes para comprar Azure, que puede ser Proveedor de soluciones en la nube (CSP), Enterprise, Empresa a través de distribuidor, GTM y Pago por uso.

Los gráficos circulares de **uso por oferta** y **uso por canal de ventas** presentan un desglose del uso por parte de las ofertas principales y los canales de ventas, respectivamente. El gráfico circular interno representa el uso sin procesar y el gráfico circular externo representa el uso normalizado.

Los gráficos circulares de **pedidos por tipo de licencia de Marketplace** y **uso por tipo de licencia de Marketplace** muestran un desglose de los pedidos y el uso por su tipo de licencia correspondiente. Los tipos de licencia incluyen:

- **Facturado a través de Azure**: Microsoft factura a los clientes en su nombre cuando decide vender su oferta a través de Microsoft con este tipo de licencia. Los tipos de pago incluyen el pago por uso con tarjeta de crédito o la facturación de Enterprise.
- **Traiga su propia licencia**: Microsoft no factura a los clientes por su uso con este tipo de oferta de Marketplace. Este uso aparece como **Obtener ahora (gratis)** en Marketplace.
- **Gratis**: Microsoft no factura a los clientes por su uso con este tipo de oferta de Marketplace. Este uso aparece como **Evaluación gratuita** en Marketplace.
- **Microsoft como revendedor**: Representa las ofertas vendidas por los revendedores de Microsoft como parte del **programa Proveedor de soluciones en la nube (CSP)** .

### <a name="customer-leaderboard"></a>Tabla de clasificación de clientes

Los 50 clientes con el mayor número de pedidos se muestran en una *tabla de clasificación*, ordenados por el mayor número de pedidos y porcentaje de pedidos.

- Seleccione un cliente para ver los detalles de su perfil, los pedidos organizados por oferta o los pedidos organizados por tipo de licencia y canal de precios de Azure.
- El gráfico de anillos **Ofertas por pedidos** presenta las cuatro principales ofertas (por recuento de pedidos) y las restantes ofertas agrupadas como "Rest all" (Todo el resto).
- El gráfico de anillos de **uso normalizado por oferta** presenta las cinco mejores ofertas por uso.

> [!NOTE]
> Solo si el cliente ha dado su consentimiento, se presentará la información personal del cliente. Puede ver esta información si ha iniciado sesión con un nivel de permisos del rol **Propietario**. Los usuarios con el rol **Colaborador** no podrán ver esta información. [Obtenga más información sobre los roles y los permisos de los usuarios](./manage-account.md#define-user-roles-and-permissions).

### <a name="seat-count-trend"></a>Tendencia de número de puestos

El gráfico de **pedidos por puesto o sitio** presenta el desglose de todos los pedidos adquiridos según el modelo de precios. El gráfico de **tendencia de número de puestos** presenta puestos frente a pedidos adquiridos para todas las ofertas de software como servicio (SaaS) por puesto.

### <a name="free-trials-saas-orders-trend"></a>Tendencia de los pedidos de SaaS de evaluación gratuita

En el gráfico de **tendencias de los pedidos de SaaS de evaluación gratuita** se muestra la tendencia de los pedidos para las ofertas de SaaS de evaluación gratuita con un período de prueba de 30 días.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los informes de análisis disponibles en el marketplace comercial del Centro de partners, consulte [Análisis del marketplace comercial en el Centro de partners](./analytics.md).
- Para información sobre los pedidos en un formato gráfico que se pueda descargar, consulte [Panel Pedidos de los análisis de marketplace comercial](./orders-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](./usage-dashboard.md).
- Para información detallada sobre los clientes, incluidas las tendencias de crecimiento, consulte [Panel Cliente de los análisis de marketplace comercial](./customer-dashboard.md).
- Para ver una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](./downloads-dashboard.md).
- Para tener una vista consolidada de comentarios de los clientes sobre las ofertas de Azure Marketplace y AppSource, consulte [Panel Calificaciones y opiniones de los análisis de marketplace comercial](./ratings-reviews.md).
- Para ver las preguntas más frecuentes sobre los análisis de marketplace comercial y un diccionario completo de términos de datos, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./faq-terminology.md).
