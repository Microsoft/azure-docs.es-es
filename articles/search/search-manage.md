---
title: Administración de servicios en el portal
titleSuffix: Azure Cognitive Search
description: Administrar un servicio de Azure Cognitive Search, un servicio hospedado de búsqueda en la nube de Microsoft Azure, mediante Azure Portal.
manager: nitinme
author: HeidiSteen
ms.author: heidist
tags: azure-portal
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/24/2020
ms.openlocfilehash: 84ddc4b427f6dc168c044f34b41e81e3b0ff19e5
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935048"
---
# <a name="service-administration-for-azure-cognitive-search-in-the-azure-portal"></a>Administración de los servicios de Azure Cognitive Search en Azure Portal

> [!div class="op_single_selector"]
>
> * [PowerShell](search-manage-powershell.md)
> * [REST API](/rest/api/searchmanagement/)
> * [SDK de .NET](/dotnet/api/microsoft.azure.management.search)
> * [Portal](search-manage.md)
> * [Python](https://pypi.python.org/pypi/azure-mgmt-search/0.1.0)> 

Azure Cognitive Search es un servicio de búsqueda basado en la nube totalmente administrado que se utiliza para la creación de una experiencia de búsqueda enriquecida en aplicaciones personalizadas. En este artículo se tratan las tareas de administración de servicios que se pueden realizar en [Azure Portal](https://portal.azure.com) para un servicio de búsqueda que ya se haya aprovisionado. Administración de servicios es ligero por diseño y se limita a las siguientes tareas:

* Compruebe el almacenamiento con el vínculo **Uso** de la página intermedia.
* Compruebe los volúmenes y la latencia de las consultas con el vínculo **Supervisión** de la página intermedia y si las solicitudes se limitaron.
* Administre el acceso con la página **Claves** que aparece a la izquierda.
* Ajuste la capacidad con la página **Escala** que aparece a la izquierda.

Las mismas tareas que se realizan en el portal también se pueden controlar mediante programación a través de las [API de Administración](/rest/api/searchmanagement/) y del [módulo Az.Search PowerShell](search-manage-powershell.md). Las tareas administrativas se representan completamente en interfaces programáticas y el portal. No hay ninguna tarea administrativa específica que esté disponible solo en una modalidad.

Azure Cognitive Search aprovecha otros servicios de Azure para realizar una supervisión y administración más profundas. Por sí solos, los únicos datos almacenados con un servicio de búsqueda son contenido (índices, indexador y definiciones de orígenes de datos, además de otros objetos). Las métricas que se indican en las páginas del portal se extraen de los registros internos en un ciclo continuo de 30 días. Para la retención de registros controlada por el usuario y eventos adicionales, necesitará [Azure Monitor](../azure-monitor/index.yml). 

## <a name="fixed-service-properties"></a>Propiedades fijas del servicio

Se determinan varios aspectos de un servicio de búsqueda cuando se aprovisiona el servicio y no se pueden cambiar más adelante:

* Nombre del servicio (no se puede cambiar el nombre de un servicio)
* Ubicación del servicio (actualmente no se puede trasladar un servicio intacto a otra región)
* Número máximo de réplicas y particiones (determinados por el nivel, ya sea Básico o Estándar)

Si empezó con el nivel Básico con el máximo de una partición y ahora necesita más particiones, deberá [crear un servicio nuevo](search-create-service-portal.md) en un nivel superior y volver a crear el contenido en el servicio nuevo. 

## <a name="administrator-rights"></a>Derechos de administrador

El aprovisionamiento o la retirada del propio servicio los puede realizar un administrador o coadministrador de una suscripción de Azure.

Con respecto al acceso al punto de conexión, cualquier persona que tenga acceso a la dirección URL del servicio y una clave de API tiene acceso al contenido. Para más información sobre las claves, consulte [Administración de las claves de API](search-security-api-keys.md).

* El acceso de solo lectura al servicio es de derechos de consulta y habitualmente se concede a una aplicación cliente al proporcionarle la dirección URL y una clave de API de consulta.
* El acceso de lectura y escritura proporciona la capacidad de agregar, eliminar o modificar objetos de servidor, como claves de API, índices, indexadores, orígenes de datos y programaciones. El acceso de lectura y escritura se concede al proporcionar la dirección URL y una clave de API de administración.

Los derechos al aparato de aprovisionamiento de servicios se conceden a través de las asignaciones de roles. El [control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/overview.md) es un sistema de autorización basado en [Azure Resource Manager](../azure-resource-manager/management/overview.md) para el aprovisionamiento de recursos de Azure. 

En el contexto de Azure Cognitive Search, las [asignaciones de roles de Azure](search-security-rbac.md) determinarán quién puede realizar tareas, independientemente de si usan el [portal](search-manage.md), [PowerShell](search-manage-powershell.md) o las [API REST de Administración](/rest/api/searchmanagement/search-howto-management-rest-api):

* Creación o eliminación de un servicio
* Escala del servicio
* Eliminación o regeneración de las claves de API
* Habilitación del registro de diagnóstico (creación de servicios)
* Habilitación de análisis de tráfico (creación de servicios)

> [!TIP]
> Mediante el uso de mecanismos de aplicación en todo el sistema de Azure, puede bloquear una suscripción o un recurso para evitar la eliminación accidental o no autorizada del servicio de búsqueda por parte de usuarios con derechos de administrador. Para más información, consulte [Bloqueo de recursos para impedir eliminación inesperada](../azure-resource-manager/management/lock-resources.md).

## <a name="logging-and-system-information"></a>Registro e información del sistema

En el nivel básico, y en los superiores, Microsoft supervisa que todos los servicios de Azure Cognitive Search tengan una disponibilidad del 99,9 % por contrato de nivel de servicio (SLA). Si el servicio es lento o el rendimiento de las solicitudes no llega a los umbrales del SLA, los equipos de soporte técnico revisan los archivos de registro a los que pueden acceder y solucionan el problema.

Azure Cognitive Search usa [Azure Monitor](../azure-monitor/index.yml) para recopilar y almacenar la actividad de indexación y consulta. Un servicio de búsqueda almacena solo su contenido (índices, definiciones de indexador, definiciones de orígenes de datos, definiciones de conjuntos de aptitudes, asignaciones de sinónimos). La información registrada y almacenamiento en caché se almacena fuera del servicio, por lo general en una cuenta de Azure Storage. Para más información sobre cómo registrar cargas de trabajo de consulta e indexación, consulte [Recopilación y análisis de datos de registro](search-monitor-logs.md).

En lo que respecta a la información general sobre el servicio, al usar solo las instalaciones integradas en Azure Cognitive Search, puede obtener información de las siguientes maneras:

* Mediante la página **Información general** del servicio, a través de notificaciones, propiedades y mensajes de estado.
* Mediante [PowerShell](search-manage-powershell.md) o la [API REST de Administración](/rest/api/searchmanagement/) para [obtener las propiedades del servicio](/rest/api/searchmanagement/services). No hay información nueva ni operaciones que se proporcionen en la capa programática. Las interfaces existen para que pueda escribir scripts.

## <a name="monitor-resource-usage"></a>Supervisar el uso de recursos

En el panel, la supervisión de recursos se limita a la información que se muestra en el panel de servicios y algunas métricas que puede obtener al consultar el servicio. En el panel de servicios, en la sección Uso, podrá determinar rápidamente si los niveles de recursos de partición son adecuados para su aplicación. Puede aprovisionar recursos externos, como la supervisión de Azure, si desea capturar y persistir eventos registrados. Para más información, consulte [Supervisión de Azure Cognitive Search](search-monitor-usage.md).

Al usar la API de REST del servicio de búsqueda, puede obtener un recuento de los documentos e índices mediante programación: 

* [Obtención de estadísticas de índice](/rest/api/searchservice/Get-Index-Statistics)
* [Documentos de recuento](/rest/api/searchservice/count-documents)

## <a name="disaster-recovery-and-service-outages"></a>Interrupciones de servicio y recuperación ante desastres

Aunque podemos recuperar los datos, Azure Cognitive Search no proporciona conmutación por error instantánea del servicio si se produce una interrupción en centro de datos o el clúster. Si se produce un error en un clúster del centro de datos, el equipo de operaciones lo detectará y trabajará para restaurar el servicio. Experimentará tiempo de inactividad durante la restauración del servicio, pero puede solicitar créditos del servicio para compensar la falta de disponibilidad de servicio según el [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/search/v1_0/). 

Si se requiera continuidad del servicio, en caso de errores catastróficos fuera de control de Microsoft, debe [aprovisionar un servicio adicional](search-create-service-portal.md) en una región diferente e implementar una estrategia de replicación geográfica para asegurarse de que los índices sean totalmente redundantes en todos los servicios.

Los clientes que usen [indizadores](search-indexer-overview.md) para rellenar y actualizar índices pueden controlar la recuperación ante desastres mediante indizadores geográficos que aprovechan el mismo origen de datos. Dos servicios en regiones diferentes, donde cada uno ejecuta un indizador, pueden indexar desde el mismo origen de datos para lograr redundancia geográfica. Si indexa desde orígenes de datos que también tienen redundancia geográfica, tenga en cuenta que los indexadores de Azure Cognitive Search solo pueden realizar una indexación incremental (fusión de actualizaciones a partir de documentos nuevos, modificados o eliminados) desde las réplicas principales. En un evento de conmutación por error, asegúrese de que el indizador apunte de nuevo a la nueva réplica principal. 

Si no utiliza indizadores, usaría el código de aplicación para enviar objetos y datos a distintos servicios de búsqueda en paralelo. Para más información, consulte [Estrategias de implementación y procedimientos recomendados de Azure Cognitive Search](search-performance-optimization.md).

## <a name="backup-and-restore"></a>Copia de seguridad y restauración

Dado que Azure Cognitive Search no es una solución de almacenamiento de datos principal, no se proporciona un mecanismo formal para realizar procesos de copias de seguridad y restauración automáticos. Sin embargo, puede usar el ejemplo de código **index-backup-restore** de este [repositorio de ejemplo .NET de Azure Cognitive Search](https://github.com/Azure-Samples/azure-search-dotnet-samples) para realizar la copia de seguridad de la definición de índice y una instantánea en una serie de archivos JSON y, después, usar esos archivos para restaurar el índice, en caso de que sea necesario. Esta herramienta también se puede usar para mover índices entre niveles de servicio.

De lo contrario, el código de aplicación que se usa para crear y rellenar un índice es la opción de restauración de facto si elimina por error un índice. Para volver a generar un índice, elimínelo (si existe), vuelva a crear el índice en el servicio y cárguelo de nuevo recuperando los datos del almacén de datos principal.

## <a name="scale-up-or-down"></a>Escalado o reducción vertical

Cada uno de los servicios de búsqueda se inicia con una cantidad mínima de una réplica y una partición. Si se registró en un [nivel que admite más capacidad](search-limits-quotas-capacity.md), haga clic en **Escala** en el panel de navegación de la izquierda para ajustar el uso de los recursos.

Al agregar una capacidad mediante cualquiera de los recursos, el servicio los usa automáticamente. No es necesario que haga nada más, pero hay un ligero retraso antes de materializarse el impacto del nuevo recurso. El aprovisionamiento de los recursos adicionales puede tardar 15 minutos, o más.

### <a name="add-replicas"></a>Adición de réplicas

La adición de réplicas permiten un aumento de las consultas por segundo (QPS) o la consecución de una alta disponibilidad. Cada réplica tiene una copia de un índice, por lo que la adición de una réplica más se traduce en que hay un índice más disponible para controlar solicitudes de consulta del servicio. Para lograr una alta disponibilidad, se requieren un mínimo de tres réplicas (consulte [Ajuste de la capacidad](search-capacity-planning.md) para más información).

Un servicio de búsqueda con más réplicas puede equilibrar la carga de solicitudes de consulta sobre un número de índices mayor. Una vez especificado un nivel del volumen de consultas, el rendimiento de las consultas va a ser más rápido cuando haya más copias del índice disponible para ofrecer la solicitud. En caso de experimentar una latencia de consulta, puede esperar que se produzca un impacto positivo en el rendimiento una vez que las réplicas adicionales estén en línea.

Aunque el rendimiento de las consultas aumente a medida que agrega réplicas, esto no lo dobla ni triplica precisamente a medida que agrega réplicas a su servicio. Todas las aplicaciones de búsqueda están sujetas a factores externos que pueden afectar al rendimiento de las consultas. Las consultas complejas y la latencia de red son dos factores que contribuyen a la existencia de variaciones en los tiempos de respuesta de las consultas.

### <a name="add-partitions"></a>Adición de particiones

Es más común agregar réplicas, pero cuando el almacenamiento está restringido, puede agregar particiones para obtener más capacidad. El nivel en el que aprovisionó el servicio determina si se pueden agregar particiones. El nivel Básico está bloqueado en una partición. El nivel Estándar y niveles superiores admiten particiones adicionales.

Las particiones se agregan en divisores de 12 (concretamente, 1, 2, 3, 4, 6 o 12). Se trata de un artefacto de particionamiento. Un índice se crea en 12 particiones de base de datos, que pueden almacenarse en su totalidad en una partición o dividirse equitativamente en 2, 3, 4, 6 o 12 particiones (una partición de base de datos por partición).

### <a name="remove-replicas"></a>Eliminación de réplicas

Tras períodos de elevados volúmenes de consultas, puede usar el control deslizante para reducir las réplicas después de que se hayan normalizado las cargas de consultas de búsqueda (por ejemplo, tras finalizar las ventas navideñas). No es necesario que haga nada más. Al reducir el recuento de réplicas, las máquinas virtuales se abandonan en el centro de datos. Ahora, sus operaciones de ingesta de consultas y datos se ejecutarán en menos VM que antes. El requisito mínimo es una réplica.

### <a name="remove-partitions"></a>Eliminación de particiones

A diferencia de la eliminación de réplicas, que no requiere que haga nada más, es posible que tenga que hacer algo si usa más almacenamiento del que se puede reducir. Por ejemplo, si su solución usa tres particiones, la reducción de su tamaño a una o dos particiones generará un error si el nuevo espacio de almacenamiento es inferior al requerido para hospedar el índice. Como cabría esperar, las opciones con las que cuenta son eliminar índices o documentos en un índice asociado para liberar espacio o mantener la configuración actual.

No existe un método de detección que indique qué particiones de índice se almacenan en particiones concretas. Cada partición proporciona un espacio de almacenamiento de aproximadamente 25 MB, de modo que será necesario reducirlo a un tamaño al que pueda ajustarse su número de particiones. Si quiere volver a una partición, las 12 particiones deberán ajustarse.

Para ayudar en una futura planificación, es posible que quiera comprobar el espacio de almacenamiento (usando [Obtención de estadísticas de índice](/rest/api/searchservice/Get-Index-Statistics)) para ver cuánto usó en realidad. 

## <a name="next-steps"></a>Pasos siguientes

* Automatización con [PowerShell](search-manage-powershell.md)

* Revisión de las técnicas de [rendimiento y optimización](search-performance-optimization.md)

* Revisión de las [características de seguridad](search-security-overview.md) para proteger contenido y operaciones

* Habilitación de [registro de diagnóstico](search-monitor-logs.md) para supervisar las cargas de trabajo de consulta e indexación