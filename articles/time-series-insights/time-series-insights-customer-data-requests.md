---
title: 'Características de las solicitudes de datos de clientes: Azure Time Series Insights | Microsoft Docs'
description: Conozca las características de las solicitudes de datos de clientes de Azure Time Series Insights.
author: deepakpalled
ms.author: dpalled
manager: cshankar
ms.date: 04/17/2020
ms.topic: conceptual
ms.service: time-series-insights
services: time-series-insights
ms.custom: seodec18
ms.openlocfilehash: 30b3832dfe3e1c4564d1d89b5c318421e48f554f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501702"
---
# <a name="summary-of-customer-data-request-features"></a>Resumen de características de solicitud de datos del cliente

Azure Time Series Insights es un servicio en la nube administrado con componentes de almacenamiento, análisis y visualización que hacen más fácil ingerir, almacenar, explorar y analizar miles de millones de eventos al mismo tiempo.

[!INCLUDE [gdpr-intro-sentence](../../includes/gdpr-intro-sentence.md)]

Para ver, exportar y eliminar datos personales que pueden estar sujetos a una solicitud de los titulares de los datos, un administrador de inquilinos de Azure Time Series Insights puede usar las API REST o Azure Portal. El uso de Azure Portal para las solicitudes del titular de los datos del servicio proporciona un método más simple para llevar a cabo estas operaciones que prefieren la mayoría de los usuarios.

## <a name="identifying-customer-data"></a>Identificación de los datos del cliente

Azure Time Series Insights considera los datos personales como datos asociados a los administradores y usuarios de Time Series Insights. Time Series Insights almacena el id. de objeto de Azure Active Directory de los usuarios con acceso al entorno. Azure Portal muestra direcciones de correo electrónico del usuario, pero no se almacenan en Time Series Insights, sino que se buscan dinámicamente mediante el id. de objeto de Azure Active Directory en Azure Active Directory.

## <a name="deleting-customer-data"></a>Eliminación de datos del cliente

Un administrador de inquilinos puede eliminar datos del cliente mediante Azure Portal.

[!INCLUDE [gdpr-dsr-and-stp-note](../../includes/gdpr-dsr-and-stp-note.md)]

Sin embargo, antes de eliminar datos del cliente a través del portal, debe quitar las directivas de acceso del usuario del entorno de Time Series Insights en Azure Portal. Para obtener más información, consulte [Concesión de acceso a los datos de un entorno de Time Series Insights mediante Azure Portal](time-series-insights-data-access.md).

También puede realizar operaciones de eliminación en las directivas de acceso mediante la API REST. Para obtener más información, lea [Directivas de acceso: eliminación](/rest/api/time-series-insights/management(gen1/gen2)/accesspolicies/delete).

Time Series Insights se integra con la hoja Directiva en Azure Portal. Tanto Time Series Insights como la hoja Directiva le permiten ver, exportar y eliminar datos del usuario almacenados en el servicio. Cualquier acción de eliminación realizada en la hoja Directiva de Azure Portal tiene como resultado la eliminación de los datos del usuario en Time Series Insights. Por ejemplo, si un usuario tiene guardada una consulta personal, esta se eliminará definitivamente del explorador de Time Series Insights. Si el usuario tiene guardada una consulta compartida, la consulta permanece, pero la información del usuario se elimina permanentemente. La nota siguiente contiene instrucciones sobre cómo realizar estas tareas.

## <a name="exporting-customer-data"></a>Exportación de datos del cliente

De forma similar a la eliminación de datos, un administrador de inquilinos puede ver y exportar los datos almacenados en Time Series Insights de la hoja Directiva en Azure Portal.

[!INCLUDE [gdpr-dsr-and-stp-note](../../includes/gdpr-dsr-and-stp-note.md)]

Si es un administrador de inquilinos, puede ver las directivas de acceso a datos del entorno de Time Series Insights en Azure Portal. Para obtener más información, consulte [Concesión de acceso a los datos de un entorno de Time Series Insights mediante Azure Portal](time-series-insights-data-access.md).

También es posible realizar operaciones de exportación de directivas de acceso mediante la operación "lista por entorno" de la API REST proporcionada. Para obtener más información, consulte [Directivas de acceso: enumerar por entorno](/rest/api/time-series-insights/management(gen1/gen2)/accesspolicies/listbyenvironment).

## <a name="to-delete-data-stored-within-time-series-insights"></a>Eliminación de los datos almacenados en Time Series Insights

Los datos personales pueden abrirse camino en el almacenamiento de Time Series Insights, un escenario distinto de los datos del administrador y del usuario. Si considera los datos almacenados en Time Series Insights como datos personales, puede exportar y eliminar dichos datos mediante los pasos siguientes:

**Visualización y exportación de datos**

Para ver y exportar datos almacenados en Time Series Insights, debe buscarlos. Puede utilizar las API de consulta o el explorador de Time Series Insights para ver y exportar datos. Para ver y exportar datos mediante el explorador de Time Series Insights, primero debe buscar los datos del usuario en cuestión. Tras la búsqueda, haga clic con el botón derecho en el gráfico y seleccione **Explorar eventos**. La cuadrícula de eventos aparece y presenta opciones para exportar los datos como CSV y JSON.

Para obtener más información, lea [Explorador de Azure Time Series Insights](time-series-insights-explorer.md).

**Eliminación de datos**

Actualmente, Time Series Insights no es compatible con la eliminación pormenorizada de datos. Sin embargo, Time Series Insights proporciona la capacidad de eliminar datos del cliente almacenados en Time Series Insights mediante la configuración de directivas de retención. Puede adaptar el período de retención de todo el entorno de Time Series Insights a cualquier número de días que se ajuste a sus requisitos de eliminación.

Para obtener más información, lea [Configuración de la retención en Time Series Insights](time-series-insights-how-to-configure-retention.md).

## <a name="next-steps"></a>Pasos siguientes

* Lea sobre cómo [conceder acceso a datos en el entorno de Time Series Insights](./time-series-insights-data-access.md).

* Vea el artículo [Explorador de Azure Time Series Insights](time-series-insights-explorer.md).

* Obtenga información sobre cómo [configurar la retención en Time Series Insights](time-series-insights-how-to-configure-retention.md).
