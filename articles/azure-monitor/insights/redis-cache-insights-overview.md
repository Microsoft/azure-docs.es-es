---
title: Azure Monitor para Azure Cache for Redis | Microsoft Docs
description: En este artículo describe la característica Azure Monitor para Azure Cache for Redis, que brinda a los propietarios de la caché una descripción rápida de los problemas de rendimiento y uso.
ms.topic: conceptual
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/10/2020
ms.openlocfilehash: e7b8a8e131cf7e0aae051f42043abf0640c927e4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90887017"
---
# <a name="explore-azure-monitor-for-azure-cache-for-redis"></a>Exploración de Azure Monitor para Azure Cache for Redis

Para todos los recursos de Azure Cache for Redis, Azure Monitor para Azure Cache for Redis (versión preliminar) proporciona una vista interactiva unificada de:

- Rendimiento general
- Errores
- Capacity
- Estado operativo

Este artículo lo ayuda a comprender las ventajas de esta experiencia de supervisión nueva. También se muestra cómo modificar y adaptar la experiencia para ajustarla a las necesidades únicas de su organización.

## <a name="introduction"></a>Introducción

Antes de comenzar la experiencia, debe entender cómo Azure Monitor para Azure Cache for Redis presenta la información de forma visual.

Ofrece:

- **Perspectiva de escala** de los recursos de Azure Cache for Redis en una ubicación única para todas las suscripciones. Puede examinar de forma selectiva solo las suscripciones y los recursos que desea evaluar.

- **Análisis de exploración en profundidad** de un recurso de Azure Cache for Redis determinado. Puede diagnosticar problemas y ver un análisis detallado de la utilización, los errores, la capacidad y las operaciones. Seleccione cualquiera de estas categorías para obtener una vista detallada de la información pertinente.  

- **Personalización** de esta experiencia, que se basa en plantillas de libro de Azure Monitor. La experiencia le permite cambiar las métricas que se muestran y modificar o establecer umbrales que se alinean con sus límites. Puede guardar los cambios en un libro personalizado y luego anclar los gráficos del libro a los paneles de Azure.

Esta característica no requiere que se habilite o configure nada. La información de Azure Cache for Redis se recopila de manera predeterminada.

>[!NOTE]
>El acceso a esta característica no se cobra, solo paga por las características básicas de Azure Monitor que configure o habilite, tal como se describe en la página de [detalles de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="view-utilization-and-performance-metrics-for-azure-cache-for-redis"></a>Visualización de las métricas de uso y rendimiento para Azure Cache for Redis

Para ver el uso y el rendimiento de las cuentas de almacenamiento en todas las suscripciones, haga los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Busque **Monitor** y seleccione **Monitor**.

    ![Cuadro de búsqueda con la palabra "Monitor" y el resultado de la búsqueda de Servicios que muestra "Monitor" con un símbolo de velocímetro](./media/cosmosdb-insights-overview/search-monitor.png)

1. Seleccione **Azure Cache for Redis**. Si esta opción no aparece, seleccione **Más** > **Azure Cache for Redis**.

### <a name="overview"></a>Información general

En **Información general**, la tabla muestra las métricas de Azure Cache for Redis interactivas. Puede filtrar los resultados en función de las opciones que seleccione en las siguientes listas desplegables:

- **Suscripciones**: solo se muestran las suscripciones que tienen un recurso de Azure Cache for Redis.  

- **Azure Cache for Redis**: puede seleccionar todo, un subconjunto o un solo recurso de Azure Cache for Redis.

- **Intervalo de tiempo**: de forma predeterminada, la tabla muestra las últimas cuatro horas de información en función de las selecciones correspondientes.

Hay un icono de contador debajo de las listas desplegables. El icono muestra el número total de recursos de Azure Cache for Redis en las suscripciones seleccionadas. Los mapas términos o códigos de color condicional para las columnas del libro informan las métricas de transacción. El color más profundo representa el valor más alto. Los colores más claros representan valores más bajos.

Al seleccionar una flecha de la lista desplegable junto a uno de los recursos de Azure Cache for Redis se revela un desglose de las métricas de rendimiento en el nivel de recurso individual.

![Captura de pantalla de la experiencia de información general](./media/redis-cache-insights-overview/overview.png)

Al seleccionar el nombre del recurso de Azure Cache for Redis resaltado en azul, verá la tabla **Información general** predeterminada para la cuenta asociada. Muestra estas columnas:

- **Memoria usada**
- **Porcentaje de memoria usada**
- **Carga del servidor**
- **Escala de tiempo de carga del servidor**
- **CPU**
- **Clientes conectados**
- **Errores de caché**
- **Errores (máx.)**

### <a name="operations"></a>Operaciones

Al seleccionar **Operaciones** en la parte superior de la página, se abre la tabla **Operaciones** de la plantilla del libro. Muestra estas columnas:

- **Operaciones totales**
- **Escala de tiempo del total de operaciones**
- **Operaciones por segundo**
- **Obtiene**
- **Conjuntos**

![Captura de pantalla de la experiencia de operaciones](./media/redis-cache-insights-overview/operations.png)

### <a name="usage"></a>Uso

Al seleccionar **Uso** en la parte superior de la página, se abre la tabla **Uso** de la plantilla del libro. Muestra estas columnas:

- **Lectura de caché**
- **Escala de tiempo de lectura de caché**
- **Escritura de caché**
- **Aciertos de caché**
- **Errores de caché**

![Captura de pantalla de la experiencia de uso](./media/redis-cache-insights-overview/usage.png)

### <a name="failures"></a>Errores

Al seleccionar **Errores** en la parte superior de la página, se abre la tabla **Errores** de la plantilla del libro. Muestra estas columnas:

- **Total de errores**
- **Conmutación por error/errores**
- **Cliente sin respuesta/errores**
- **RDB/errores**
- **AOF/errores**
- **Exportación/errores**
- **Pérdida de datos/errores**
- **Importación/errores**

![Captura de pantalla de errores con un desglose por tipo de solicitud HTTP](./media/redis-cache-insights-overview/failures.png)

### <a name="metric-definitions"></a>Definiciones de métricas

Para una lista completa de las definiciones de métricas que conforman estos libros, consulte el [artículo sobre las métricas disponibles y los intervalos de informes](../../azure-cache-for-redis/cache-how-to-monitor.md#available-metrics-and-reporting-intervals).

## <a name="view-from-an-azure-cache-for-redis-resource"></a>Vista desde un recurso de Azure Cache for Redis

Para acceder a Azure Monitor para Azure Cache for Redis directamente desde un recurso individual:

1. En Azure Portal, seleccione Azure Cache for Redis.

2. En la lista, elija un recurso individual de Azure Cache for Redis. En la sección de supervisión, seleccione Información.

    ![Captura de pantalla de las opciones de menú con las palabras "Información" resaltadas en un cuadro rojo.](./media/redis-cache-insights-overview/insights.png)

También se puede acceder a estas vistas seleccionando el nombre del recurso de un recurso de Azure Cache for Redis en el libro de nivel de Azure Monitor.

### <a name="resource-level-overview"></a>Información general de nivel de recurso

En el libro **Información general** de Azure Cache for Redis, se muestran varias métricas de rendimiento que le dan acceso a:

- Gráficos interactivos de rendimiento que muestran los detalles más esenciales relacionados con el rendimiento de Azure Cache for Redis.

- Iconos de estados y métricas que resaltan el rendimiento de las particiones, el total de clientes conectados y la latencia total.

![Captura de pantalla del panel de información general con información sobre el rendimiento de la CPU, la memoria usada, los clientes conectados, los errores, las claves expiradas y las claves expulsadas](./media/redis-cache-insights-overview/resource-overview.png)

Al seleccionar cualquiera de las demás pestañas para **Rendimiento** u **Operaciones** se abren los libros correspondientes.

### <a name="resource-level-performance"></a>Rendimiento de nivel de recurso

![Captura de pantalla de los gráficos de rendimiento de recursos](./media/redis-cache-insights-overview/resource-performance.png)

### <a name="resource-level-operations"></a>Operaciones de nivel de recurso

![Captura de pantalla de los gráficos de operaciones de recursos](./media/redis-cache-insights-overview/resource-operations.png)

## <a name="pin-export-and-expand"></a>Anclaje, exportación y expansión

Para anclar cualquier sección de métrica a un [panel de Azure](../../azure-portal/azure-portal-dashboards.md), seleccione el símbolo de chincheta en la esquina superior derecha de la sección.

![Una sección de métrica con el símbolo de chincheta resaltado](./media/cosmosdb-insights-overview/pin.png)

Para exportar los datos a un formato de Excel, seleccione el símbolo de flecha abajo que está a la izquierda del símbolo de chincheta.

![Un símbolo de exportación de libro resaltado](./media/cosmosdb-insights-overview/export.png)

Para expandir o contraer todas las vistas de un libro, seleccione el símbolo de expansión que se encuentra a la izquierda del símbolo de exportación.

![Un símbolo de expansión de libro resaltado](./media/cosmosdb-insights-overview/expand.png)

## <a name="customize-azure-monitor-for-azure-cache-for-redis"></a>Personalización de Azure Monitor para Azure Cache for Redis

Como esta experiencia se basa en plantillas de libros de Azure Monitor, puede seleccionar **Personalizar** > **Editar** > **Guardar** para guardar una copia de la versión modificada en un libro personalizado.

![Una barra de comandos con Personalizar resaltado](./media/cosmosdb-insights-overview/customize.png)

Los libros se guardan dentro de un grupo de recursos ya sea en la sección **Mis informes** o en la sección **Informes compartidos**. **Mis informes** solo está disponible para usted. **Informes compartidos** está disponible para todos los usuarios con acceso al grupo de recursos.

Después de guardar un libro personalizado, vaya a la galería de libros para abrirlo.

![Una barra de comandos con la Galería resaltada](./media/cosmosdb-insights-overview/gallery.png)

## <a name="troubleshooting"></a>Solución de problemas

Para obtener instrucciones para la solución de problemas, consulte el [artículo dedicado de solución de problemas](troubleshoot-workbooks.md) de conclusiones basadas en libros.

## <a name="next-steps"></a>Pasos siguientes

* Configure [alertas de métricas](../platform/alerts-metric.md) y [notificaciones de estado del servicio](../../service-health/alerts-activity-log-service-notifications-portal.md) para configurar alertas automáticas que ayuden a detectar los problemas.

* Conozca los escenarios compatibles con libros, cómo crear o personalizar informes y muchos otros temas en el artículo sobre cómo [crear informes interactivos con libros de Azure Monitor](../platform/workbooks-overview.md).
