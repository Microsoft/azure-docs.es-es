---
title: Supervisión de operaciones y actividades
titleSuffix: Azure Cognitive Search
description: Habilite el registro y obtenga métricas de actividades de consulta, uso de recursos y otros datos del sistema de un servicio de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: d47f6c20246e3210b58dbc9c802a11c866ae305e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935014"
---
# <a name="monitor-operations-and-activity-of-azure-cognitive-search"></a>Supervisión de operaciones y actividades de Azure Cognitive Search

Este artículo es una introducción a los conceptos y herramientas de supervisión de Azure Cognitive Search. Para la supervisión holística, puede usar una combinación de funcionalidad integrada y servicios complementarios como Azure Monitor.

En general puede realizar el seguimiento de lo siguiente:

* Servicio: estado o disponibilidad, y cambios en la configuración del servicio.
* Almacenamiento: usado y disponible, con recuentos para cada tipo de contenido en relación con la cuota permitida para el nivel de servicio.
* Actividad de consulta: volumen, latencia y consultas limitadas o eliminadas. Las solicitudes de consulta registradas requieren [Azure Monitor](#add-azure-monitor).
* Actividad de indexación: requiere [registro de diagnóstico](#add-azure-monitor) con Azure Monitor.

Un servicio de búsqueda no admite la autenticación por usuario, por lo que no se encontrará información de identidad en los registros.

## <a name="built-in-monitoring"></a>Supervisión integrada

La supervisión integrada hace referencia a las actividades registradas por un servicio de búsqueda. A excepción del diagnóstico, para este nivel de supervisión no es necesaria ninguna configuración.

Azure Cognitive Search mantiene los datos internos en una programación gradual de 30 días para notificar el estado del servicio y las métricas de consulta, que puede encontrar en el portal o a través de estas [API REST](#monitoring-apis).

La captura de pantalla siguiente le ayuda a buscar la información de supervisión en el portal. Los datos estarán disponibles tan pronto como empiece a usar el servicio. Las páginas del portal se actualizan cada pocos minutos.

* En la pestaña **Supervisión** de la página Información general principal se muestran el volumen de la consulta, la latencia y si el servicio está bajo presión.
* **Registro de actividad**, en el panel de navegación de la izquierda, está conectado a Azure Resource Manager. El registro de actividad informa sobre las acciones emprendidas por Resource Manager: disponibilidad y estado del servicio, cambios en la capacidad (réplicas y particiones), y actividades relacionadas con las claves de API.
* Los valores de **Supervisión** (más abajo) proporcionan alertas, métricas y registros de diagnóstico configurables. Créelos cuando los necesite. Una vez que los datos se recopilan y almacenan, puede consultar o visualizar los detalles de la información.

![Integración de Azure Monitor en un servicio de búsqueda](./media/search-monitor-usage/azure-monitor-search.png
 "Integración de Azure Monitor en un servicio de búsqueda")

> [!NOTE]
> Como las páginas del portal se actualizan cada pocos minutos, los números que se notifican son aproximados y están pensados para proporcionarle una idea general de la capacidad del sistema para atender las solicitudes. Las métricas actuales, como las consultas por segundo (QPS), pueden ser mayores o menores que el número que se muestra en la página. Si la precisión es un requisito, considere la posibilidad de usar las API.

<a name="monitoring-apis"> </a>

### <a name="apis-useful-for-monitoring"></a>API útiles para la supervisión

Puede usar las API siguientes para recuperar la misma información que se encuentra en las pestañas Supervisión y Uso del portal.

* [GET Service Statistics](/rest/api/searchservice/get-service-statistics)
* [GET Index Statistics](/rest/api/searchservice/get-index-statistics)
* [GET Document Counts](/rest/api/searchservice/count-documents)
* [GET Indexer Status](/rest/api/searchservice/get-indexer-status)

### <a name="activity-logs-and-service-health"></a>Registros de actividad y estado del servicio

En la página [**Registro de actividad**](../azure-monitor/platform/activity-log.md#view-the-activity-log) del portal se recopila información de Azure Resource Manager y se notifican los cambios en el estado del servicio. Puede supervisar si en el registro de actividad hay condiciones críticas, de error y de advertencia relacionadas con el estado del servicio.

Entre las entradas comunes se incluyen referencias a las claves de API: notificaciones de información genéricas como *Obtener la clave de administrador* y *Obtener claves de consulta*. Estas actividades indican las solicitudes realizadas con la clave de administración (creación o eliminación de objetos) o la clave de consulta, pero no muestran la propia solicitud. Para obtener información de este nivel de detalle, debe configurar el registro de diagnóstico.

Puede acceder al **registro de actividad** desde el panel de navegación izquierdo, desde las notificaciones de la barra de comandos de la ventana superior o desde la página **Diagnose and solve problems** (Diagnosticar y resolver problemas).

### <a name="monitor-storage-in-the-usage-tab"></a>Supervisión del almacenamiento en la pestaña Uso

Para la supervisión visual en el portal, la pestaña **Uso** muestra la disponibilidad de los recursos, en relación con los [límites](search-limits-quotas-capacity.md) actuales que impone el nivel de servicio. Si va a concretar las decisiones sobre [qué nivel usar para las cargas de trabajo de producción](search-sku-tier.md) o si [ajustar el número de réplicas activas y particiones](search-capacity-planning.md), estas métricas pueden ayudarle con esas decisiones al mostrarle la velocidad a la que se consumen los recursos y el modo en que la configuración actual administra la carga existente.

La siguiente ilustración es del servicio gratuito, que está limitado a 3 objetos de cada tipo y 50 MB de almacenamiento. Un servicio básico o estándar tiene límites más altos y, si aumentan los recuentos de particiones, el almacenamiento máximo aumenta proporcionalmente.

![Estado de uso en relación con los límites del nivel](./media/search-monitor-usage/usage-tab.png
 "Estado de uso en relación con los límites del nivel")

> [!NOTE]
> Las alertas relacionadas con el almacenamiento no están disponibles actualmente; el consumo de almacenamiento no se agrega ni se registra en la tabla **AzureMetrics** de Azure Monitor. Para obtener alertas de almacenamiento, tendría que [compilar una solución personalizada](../azure-monitor/insights/solutions.md) que emita notificaciones relacionadas con los recursos, en las que el código comprueba el tamaño del almacenamiento y controla la respuesta.

<a name="add-azure-monitor"></a>

## <a name="add-on-monitoring-with-azure-monitor"></a>Supervisión complementaria con Azure Monitor

Muchos servicios, entre los que se incluye Azure Cognitive Search, se integran con [Azure Monitor](../azure-monitor/index.yml) para ofrecer alertas adicionales, métricas y datos de diagnóstico de registro. 

[Habilite el registro de diagnóstico](search-monitor-logs.md) para un servicio de búsqueda si quiere controlar la recopilación y el almacenamiento de los datos. Los eventos registrados capturados por Azure Monitor se almacenan en la tabla **AzureDiagnostics** y se componen de datos operativos relacionados con las consultas y la indexación.

Azure Monitor proporciona varias opciones de almacenamiento y la que seleccione determina cómo puede consumir los datos:

* Elija Almacenamiento de blobs de Azure si quiere [visualizar datos de registro](search-monitor-logs-powerbi.md) en un informe de Power BI.
* Elija Log Analytics si quiere explorar los datos a través de consultas de Kusto.

Azure Monitor tiene su propia estructura de facturación y los registros de diagnóstico a los que se hace referencia en esta sección tienen un costo asociado. Para obtener más información, vea [Uso y costos estimados en Azure Monitor](../azure-monitor/platform/usage-estimated-costs.md).

## <a name="monitor-user-access"></a>Supervisión del acceso de los usuarios

Dado que los índices de búsqueda son un componente de una aplicación cliente mayor, no hay ninguna metodología integrada para controlar o supervisar el acceso a un índice por usuario. Se asume que las solicitudes provienen de una aplicación cliente, tanto para las solicitudes de consulta como del administrador. Las operaciones de escritura y lectura del administrador incluyen la creación, actualización y eliminación de objetos en todo el servicio. Las operaciones de solo lectura son consultas a la colección de documentos, cuyo ámbito es un solo índice. 

De por sí, lo que verá en los registros de actividad son referencias a llamadas mediante claves de administrador o claves de consulta. La clave apropiada se incluye en las solicitudes que parten del código de cliente. El servicio no está equipado para administrar los tokens de identidad o la suplantación.

Cuando existen requisitos empresariales para la autorización por usuario, se recomienda la integración con Azure Active Directory. Puede usar $filter y las identidades del usuario para [recortar los resultados de la búsqueda](search-security-trimming-for-azure-search-with-aad.md) de documentos que un usuario no debe ver. 

No hay ninguna forma de registrar esta información por separado de la cadena de consulta que incluye el parámetro $filter. Para obtener información acerca de la generación de informes de cadenas de consulta, consulte [Administración de consultas](search-monitor-queries.md).

## <a name="next-steps"></a>Pasos siguientes

La fluidez con Azure Monitor es esencial para supervisar todos los servicios de Azure, lo que incluye recursos como Azure Cognitive Search. Si no conoce Azure Monitor, dedique un tiempo a leer los artículos relacionados con los recursos. Además de los tutoriales, el siguiente artículo es un buen punto de partida.

> [!div class="nextstepaction"]
> [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/insights/monitor-azure-resource.md)