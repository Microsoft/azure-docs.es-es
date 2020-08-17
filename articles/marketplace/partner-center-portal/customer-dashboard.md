---
title: Panel Cliente de los análisis del marketplace comercial del Centro de partners, Azure Marketplace y Microsoft AppSource
description: Obtenga detalles sobre cómo acceder a la información de los clientes, incluidas las tendencias de crecimiento, mediante el panel Cliente de los análisis de marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 07/22/2020
author: shganesh-dev
ms.author: shganesh
ms.openlocfilehash: 44e992bb1445e1e58f42ec150a8e2c0682eea98b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317593"
---
# <a name="customer-dashboard-in-commercial-marketplace-analytics"></a>Panel Cliente de los análisis de marketplace comercial

En este artículo se proporciona información sobre el **panel Cliente** del Centro de partners. En este panel se muestra información sobre los clientes, incluidas las tendencias de crecimiento, presentada en un formato gráfico y descargable

Para acceder al **panel Cliente**, abra el panel **[Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)** en Marketplace comercial.

>[!NOTE]
> Para obtener definiciones detalladas de la terminología de análisis, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./faq-terminology.md).

## <a name="customer-dashboard"></a>Panel Cliente

En el **panel Cliente** del menú **Analizar** se muestran los datos de los clientes que han adquirido sus ofertas. Puede ver representaciones gráficas de los elementos siguientes:

- [Resumen de clientes](#customer-summary)
- [Clientes por geografía](#customer-by-geography)
- [Tendencias de clientes](#customer-trends)
- [Clientes por pedidos o uso](#customers-by-orders-and-usage)
- [Clientes por planes](#customers-by-plans)
- [Pedidos y uso por tipo de cliente](#orders-and-usage-by-customer-type)
- [Tabla de detalles de clientes](#customer-details-table)
- [Filtros de la página Clientes](#customer-page-filters)

La latencia máxima entre la adquisición de clientes y la generación de informes en el Centro de partners es de 48 horas.

### <a name="customer-summary"></a>Resumen de clientes

En la sección Resumen de clientes se muestra un recuento de todos los clientes, incluidos los nuevos, existentes y perdidos, durante el intervalo de fechas seleccionado.

- El valor de Clientes totales se define como el recuento de todos los clientes que han comprado su oferta y tienen al menos un pedido que no se ha cancelado.
- El porcentaje de crecimiento de los clientes en comparación con el mes anterior se indica mediante el número y el indicador hacia arriba en verde o hacia abajo en rojo.
- Las tendencias de crecimiento se representan mediante gráficos de barras y mostrarán el valor de cada mes al mantener el puntero sobre las columnas del gráfico.

Hay tres **tipos de clientes**: nuevos, existentes y perdidos.

- Un nuevo cliente ha adquirido una o varias de sus ofertas por primera vez en el mes seleccionado.
- Un cliente existente ha adquirido una o varias de sus ofertas antes del mes seleccionado.
- Un cliente perdido ha cancelado todas las ofertas previamente adquiridas.

### <a name="customer-by-geography"></a>Clientes por geografía

En el gráfico **Clientes por geografía** se muestran los recuentos de todos los clientes y los clientes adquiridos durante el intervalo de fechas seleccionado, que se asignan en función del país o región del cliente. El tono del color de claro a oscuro del mapa representa el número de clientes de bajo a alto. Seleccione un registro de la tabla para acercar un país o región.

En el mapa térmico se muestra el número de clientes y el porcentaje por país o región del cliente. Puede mover el mapa para ver la ubicación exacta y hacer zoom en una ubicación específica. Este mapa tiene una cuadrícula complementaria que le permite ver el porcentaje de clientes por ubicación, así como los clientes que se agregaron recientemente a esa ubicación.

### <a name="customer-trends"></a>Tendencias de clientes

El gráfico de **tendencias de clientes** muestra el recuento de todos los clientes, incluidos nuevos, existentes y perdidos, con una tendencia de crecimiento mes a mes.

- El gráfico de líneas representa los porcentajes de crecimiento globales del cliente.
- La columna de mes representa el número de clientes apilados por clientes nuevos, existentes y perdidos.
- El recuento de clientes perdidos se muestra en la dirección negativa del eje Y.
- Puede seleccionar elementos específicos de la leyenda para mostrar vistas más detalladas. Por ejemplo, seleccione los nuevos clientes en la leyenda para mostrar solo los nuevos clientes.
- Puede usar el control deslizante de la parte superior del gráfico para desplazarse hacia la derecha y la izquierda a lo largo del eje x y centrarse en puntos de datos específicos para ver información más detallada.
- Al mantener el puntero sobre una columna del gráfico aparecerán los detalles de solo ese mes.

### <a name="customers-by-orders-and-usage"></a>Clientes por pedidos o uso

El gráfico **Clientes por pedido/uso** tiene las tres pestañas siguientes: "pedidos", "uso normalizado" y "uso sin procesar". El **percentil de principales clientes** muestra el eje x, según lo determina el número de pedidos. El eje y muestra el número de pedidos del cliente. En el eje z secundario (gráfico de líneas) se muestra el porcentaje acumulado del número total de pedidos. Puede mostrar detalles manteniendo el puntero sobre los puntos del gráfico de líneas.

A modo de ejemplo, vea el siguiente gráfico para ver el uso normalizado: El 30.° percentil superior de clientes contribuye al 87 % del uso normalizado de forma acumulativa. El 30.º percentil de clientes solo contribuye con 1,57 millones de horas de uso.

### <a name="customers-by-plans"></a>Clientes por planes

A continuación, se describen los gráficos **Customers by SKUs/usage** (Clientes por planes/uso).

1. En la tabla de clasificación se presentan los detalles de los 50 clientes principales clasificados por recuento de pedidos. Después de seleccionar un cliente, los detalles de este se muestran en las secciones 2, 3 y 4 de esta tabla de clasificación.
2. Los detalles del perfil Cliente se muestran en este espacio cuando los editores inician sesión con un rol de propietario. Si los editores inician sesión con un rol de colaborador, los detalles de esta sección no estarán disponibles.
3. En el gráfico de anillos **Pedidos por planes** se muestra el desglose de los pedidos realizados de planes. Se muestran los cinco planes principales con el número de pedidos más alto, mientras que el resto de los pedidos se agrupan en "Rest all" ("Todo el resto").
4. En el gráfico de anillos **Seats by plans** (Puestos por planes) se muestra el desglose de los puestos pedidos de planes. Se muestran los cinco planes principales con el número de puestos más alto, mientras que el resto de los pedidos se agrupan en "Rest all" ("Todo el resto").

### <a name="orders-and-usage-by-customer-type"></a>Pedidos y uso por tipo de cliente

En el gráfico **Orders/usage by customer type** (Pedidos/uso por tipo de cliente) se muestra el número de pedidos, el uso normalizado y las horas de uso sin procesar divididas entre nuevos clientes y clientes existentes. Estos se muestran respectivamente en dicho gráfico en función de la selección de pedidos y el uso normalizado y sin procesar.

- Un nuevo cliente ha adquirido una o varias de sus ofertas o ha notificado el uso por primera vez en el mismo mes natural (eje y).
- Un cliente existente ha adquirido una de sus ofertas con anterioridad o ha notificado el uso antes del mes natural especificado (en el eje y).

### <a name="customer-details-table"></a>Tabla de detalles de clientes

En la tabla **Detalles del cliente** se muestra una lista numerada de los primeros 1000 clientes ordenados por la fecha en que adquirieron una de sus ofertas.

- La información personal del cliente estará disponible solo si el cliente ha dado su consentimiento. Solo puede ver esta información si ha iniciado sesión con un nivel de rol de propietario de permisos. Obtenga más información sobre los roles y los permisos de los usuarios.
- Cada columna de la cuadrícula se puede ordenar.
- Los datos se pueden extraer en un archivo TSV si el recuento de los registros es inferior a 1000.
- Si el número de registros es superior a 1000, los datos exportados se colocarán de forma asincrónica en una página de descargas durante los próximos 30 días.
- Aplique filtros a la tabla para mostrar solo los datos que le interesen. Filtre los datos por nombre de empresa, identificador de cliente, identificador de suscripción de Marketplace, tipo de licencia de Azure, fecha de adquisición, fecha de pérdida, correo electrónico del cliente, país/estado/ciudad/código postal del cliente, idioma del cliente, etc.
- Cuando un cliente protegido adquiere una oferta, la información de **Customer Detailed Data** (Datos detallados del cliente) se enmascara (************).
- Los detalles de la dimensión del cliente, como el nombre de la empresa, el nombre del cliente y el correo electrónico del cliente, están en el nivel de identificador de la organización, no en el nivel de transacción de Azure Marketplace o AppSource.

### <a name="customer-page-filters"></a>Filtros de la página Clientes

Los filtros de la página **Clientes** se aplican a nivel de la página Clientes. Puede seleccionar varios filtros para representar el gráfico para los criterios que quiera ver, así como los datos que quiera que se muestren en la cuadrícula o exportación "Datos detallados de los pedidos". Los filtros se aplican a los datos extraídos del intervalo de datos que ha seleccionado en la esquina superior derecha de la página de pedidos.

>[!NOTE]
> Las definiciones detalladas de cada uno de los campos de la cuadrícula Cliente, los filtros de página y sus selecciones posibles se encuentran en el artículo de preguntas frecuentes indicado a continuación.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los informes de análisis disponibles en el marketplace comercial del Centro de partners, consulte [Análisis del marketplace comercial en el Centro de partners](./analytics.md).
- Para ver los gráficos, las tendencias y los valores de datos agregados que resuman la actividad de la oferta en el marketplace, consulte [Panel Resumen de los análisis de marketplace comercial](./summary-dashboard.md).
- Para información sobre los pedidos en un formato gráfico que se pueda descargar, consulte [Panel Pedidos de los análisis de marketplace comercial](./orders-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](./usage-dashboard.md).
- Para obtener una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](./downloads-dashboard.md).
- Para tener una vista consolidada de comentarios de los clientes sobre las ofertas de Azure Marketplace y AppSource, consulte [Panel Calificaciones y opiniones de los análisis de marketplace comercial](./ratings-reviews.md).
- Para ver las preguntas más frecuentes sobre los análisis de marketplace comercial y un diccionario completo de términos de datos, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./faq-terminology.md).
