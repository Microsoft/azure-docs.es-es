---
title: Creación de un servicio de búsqueda en el portal
titleSuffix: Azure Cognitive Search
description: En este inicio rápido del portal, aprenderá a configurar un recurso de Azure Cognitive Search en Azure Portal. Elija grupos de recursos, regiones, SKU o planes de tarifa.
manager: nitinme
author: tchristiani
ms.author: terrychr
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 07/14/2020
ms.openlocfilehash: b9507e500282afbdfba5cedd5420974aa8b3ade8
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87554029"
---
# <a name="quickstart-create-an-azure-cognitive-search-service-in-the-portal"></a>Inicio rápido: Creación de un servicio Azure Cognitive Search en el portal

Azure Cognitive Search es un recurso independiente que se usa para agregar una experiencia de búsqueda a las aplicaciones personalizadas. Cognitive Search se integra fácilmente con otros servicios de Azure, con aplicaciones de servidores de red o con software que se ejecuta en otras plataformas de la nube.

En este artículo, aprenderá a crear un recurso en [Azure Portal](https://portal.azure.com/).

[![GIF animado](./media/search-create-service-portal/AnimatedGif-AzureSearch-small.gif)](./media/search-create-service-portal/AnimatedGif-AzureSearch.gif#lightbox)

¿Prefiere PowerShell? Use la [plantilla de servicio](https://azure.microsoft.com/resources/templates/101-azure-search-create/) de Azure Resource Manager. Para obtener ayuda para empezar, consulte [Administración de Azure Cognitive Search con PowerShell](search-manage-powershell.md).

## <a name="before-you-start"></a>Antes de comenzar

Las siguientes propiedades de servicio son fijas para la vigencia del servicio y su cambio requiere un nuevo servicio. Dado que son fijas, tenga en cuenta las implicaciones de uso al rellenar cada propiedad:

* El nombre del servicio pasa a formar parte del punto de conexión de la dirección URL ([examine las sugerencias](#name-the-service) para elegir nombres de servicio útiles).
* El nivel de servicio [afecta a la facturación](search-sku-tier.md) y establece un límite superior en la capacidad. Algunas características no están disponibles en el nivel gratuito.
* La región del servicio puede determinar la disponibilidad de ciertos escenarios. Si necesita [características de alta seguridad](search-security-overview.md) o [enriquecimiento con IA](cognitive-search-concept-intro.md), Azure Cognitive Search deberá estar en la misma región que los restantes servicios, o bien en aquellas regiones que proporcionen la característica en cuestión. 

## <a name="subscribe-free-or-paid"></a>Suscripción (gratuita o de pago)

[Abra una cuenta gratuita de Azure ](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F) y use créditos gratuitos para probar servicios de pago de Azure. Cuando se consuman los créditos, mantenga la cuenta y siga usando servicios de Azure gratuitos, como Websites. No se le realizará ningún cargo en su tarjeta de crédito a menos que cambie explícitamente la configuración y lo solicite.

Opcionalmente, [puede activar los beneficios de suscriptores de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F). Una suscripción a MSDN le proporciona créditos todos los meses que puede usar para servicios de Azure de pago. 

## <a name="find-azure-cognitive-search"></a>Búsqueda de Azure Cognitive Search

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Haga clic en el signo más ("+ Crear recurso") en la esquina superior izquierda.

1. Escriba "Azure Cognitive Search" en la barra de búsqueda o vaya al recurso a través de **Web** > **Azure Cognitive Search**.

![Creación de un recurso en el portal](./media/search-create-service-portal/find-search3.png "Creación de un recurso en el portal")

## <a name="choose-a-subscription"></a>Elija una suscripción

Si tiene más de una suscripción, elija una para el servicio de búsqueda. Si va a implementar el [cifrado doble](search-security-overview.md#double-encryption), o cualesquiera otras características que dependan de identidades de servicio administradas, elija la misma suscripción que la que se usó para Azure Key Vault, u otros servicios para los que se usen identidades administradas.

## <a name="set-a-resource-group"></a>Configuración de un grupo de recursos

Un grupo de recursos es un contenedor que almacena los recursos relacionados con una solución de Azure. Es obligatorio para el servicio de búsqueda. También es útil para administrar todos los recursos, incluidos los costos. Un grupo de recursos puede estar formado por un servicio, o bien por varios servicios que se usen de forma conjunta. Por ejemplo, si usa Azure Cognitive Search para indexar una base de datos de Azure Cosmos DB, puede hacer que ambos servicios formen parte de un mismo grupo de recursos con fines de administración. 

Si no combina recursos para formar un solo grupo o si los grupos de recursos existentes se rellenan con los recursos usados en soluciones no relacionadas, cree un grupo de recursos solo para su recurso de Azure Cognitive Search. 

![Creación de un nuevo grupo de recursos](./media/search-create-service-portal/new-resource-group.png "Creación de un nuevo grupo de recursos")

Con el tiempo, puede realizar un seguimiento de todos los costos actuales y previstos, o bien puede ver los cargos de los recursos individuales. En la captura de pantalla siguiente se muestra el tipo de información de costos que puede esperar ver al combinar varios recursos en un solo grupo.

![Administración de costos en el nivel del grupo de recursos](./media/search-create-service-portal/resource-group-cost-management.png "Administración de costos en el nivel del grupo de recursos")

> [!TIP]
> Los grupos de recursos simplifican la limpieza porque, al eliminar uno de ellos, se eliminan también todos los servicios que contiene. En el caso de proyectos de prototipo que usan muchos servicios, si se ponen todos ellos en el mismo grupo de recursos, la limpieza resulta más fácil después de que el proyecto ha finalizado.

## <a name="name-the-service"></a>Asignación de un nombre al servicio

En Detalles de la instancia, proporcione un nombre de servicio en el campo **URL**. Un nombre de servicio forma parte del punto de conexión de la dirección URL con que se emiten llamadas API: `https://your-service-name.search.windows.net`. Por ejemplo, si quiere que el punto de conexión sea `https://myservice.search.windows.net`, debe escribir `myservice`.

Requisitos de nombre de servicio:

* Debe ser único dentro del espacio de nombres search.windows.net.
* Debe tener una longitud que oscile entre 2 y 60 caracteres.
* Se deben usar letras minúsculas, números o guiones ("-").
* No se deben usar guiones ("-") en los dos primeros caracteres ni en el último.
* No se pueden usar guiones consecutivos ("--").

> [!TIP]
> Si piensa que va a usar varios servicios, le recomendamos que incluya la región (o ubicación) en el nombre del servicio como una convención de nomenclatura. Los servicios que estén en la misma región pueden intercambiar datos sin costo alguno; por lo que, si Azure Cognitive Search se encuentra en la región Oeste de EE. UU. y tiene otros servicios en esta misma región, si usa un nombre como `mysearchservice-westus`, no tendrá que ir a la página de propiedades al decidir cómo combinar o conectar recursos.

## <a name="choose-a-location"></a>Selección de una ubicación

Azure Cognitive Search no está disponible en la mayoría de las regiones. Se puede encontrar la lista de regiones admitidas en la [página de precios](https://azure.microsoft.com/pricing/details/search/).

> [!Note]
> Centro de la India y Norte de Emiratos Árabes Unidos no están disponibles actualmente para los nuevos servicios. En el caso de los servicios que ya están disponibles en esas regiones, puede escalar verticalmente sin restricciones, y el servicio se admite completamente en dicha región. Las restricciones son temporales y se limitan solo a los nuevos servicios. Se eliminará esta nota cuando ya no se apliquen las restricciones.
>
> El cifrado doble solo está disponible en ciertas regiones. Para más información, consulte [Cifrado doble](search-security-overview.md#double-encryption).

### <a name="requirements"></a>Requisitos

 Si usa el enriquecimiento con IA, cree el servicio de búsqueda en la misma región que Cognitive Services. *La coubicación de Azure Cognitive Search y Cognitive Services en la misma región es un requisito para el enriquecimiento con inteligencia artificial*.

 Los clientes con requisitos de continuidad empresarial y recuperación ante desastres (BCDR) deben crear sus servicios en [pares regionales](https://docs.microsoft.com/azure/best-practices-availability-paired-regions#azure-regional-pairs). Por ejemplo, si está trabajando en Norteamérica, puede elegir Este de EE. UU. y Oeste de EE. UU., o Centro-norte de EE. UU. y Centro-sur de EE. UU., para cada servicio.

### <a name="recommendations"></a>Recomendaciones

Si usa varios servicios de Azure, elija una región que también hospede su servicio de datos o de aplicación. De este modo, se minimizan o anulan los cargos de ancho de banda de los datos salientes (no se aplican cargos por los datos salientes cuando los servicios se encuentran en la misma región).

## <a name="choose-a-pricing-tier-sku"></a>Selección de un plan de tarifa (SKU)

[Azure Cognitive Search se ofrece actualmente en varios planes de tarifa](https://azure.microsoft.com/pricing/details/search/): Gratis, Básico o Estándar. Cada plan tiene su propia [capacidad y sus propios límites](search-limits-quotas-capacity.md). Consulte [Selección SKU o plan de tarifa](search-sku-tier.md) para obtener instrucciones.

Básico y Estándar son las opciones más comunes para las cargas de trabajo de producción, pero la mayoría de los clientes empiezan con el servicio gratuito. Las diferencias principales entre los niveles son el tamaño y la velocidad de las particiones, así como los límites en el número de objetos que se pueden crear.

Recuerde que una vez que se crea el servicio, el plan de tarifa no se puede cambiar. Si necesita un nivel superior o inferior, deberá volver a crear el servicio.

## <a name="create-your-service"></a>Creación del servicio

Después de proporcionar los datos necesarios, ya puede crear el servicio. 

![Revisar y crear el servicio](./media/search-create-service-portal/new-service3.png "Revisar y crear el servicio")

El servicio se implementa en cuestión de minutos. El progreso se puede supervisar mediante las notificaciones de Azure. Considere la posibilidad de anclar el servicio al panel para facilitar el acceso en el futuro.

![Supervisar y ajustar el servicio](./media/search-create-service-portal/monitor-notifications.png "Supervisar y ajustar el servicio")

## <a name="get-a-key-and-url-endpoint"></a>Obtención de una clave y un punto de conexión de dirección URL

Excepto en el caso de que use el portal, para acceder mediante programación al nuevo servicio, tendrá que especificar el punto de conexión de la dirección URL y una clave de API de autenticación.

1. En la página **Información general**, busque y copie el punto de conexión de dirección URL en el lado derecho de la página.

2. En la página **Claves**, copie una de las claves de administración (son equivalentes). Las claves de API de administrador son necesarias para crear, actualizar y eliminar objetos en el servicio. Por el contrario, las claves de consulta proporcionan acceso de lectura al contenido del índice.

   ![Página de información general del servicio con punto de conexión de dirección URL](./media/search-create-service-portal/get-url-key.png "Punto de conexión de dirección URL y otros detalles del servicio")

Para las tareas basadas en el portal, no se necesita un punto de conexión y una clave. El portal ya está vinculado a un recurso de Azure Cognitive Search con derechos de administrador. Para un tutorial del portal, empiece por [Inicio rápido: Creación de un índice de Azure Cognitive Search en el portal](search-get-started-portal.md).

## <a name="scale-your-service"></a>Escalar el servicio

Después de aprovisionado el servicio, se puede escalar para satisfacer sus necesidades. Si ha elegido el nivel Estándar para el servicio Azure Cognitive Search, puede escalar el servicio en dos dimensiones: réplicas y particiones. Si ha elegido el nivel Básico, solo puede agregar réplicas. Si ha aprovisionado el servicio gratuito, el escalado no está disponible.

Las ***particiones*** permiten que el servicio almacene y busque en más documentos.

Las ***réplicas*** permiten al servicio administrar una carga más elevada de consultas de búsqueda.

La incorporación de recursos aumenta la factura mensual. La [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) puede ayudarle a entender cómo repercute la incorporación de recursos en la facturación. Recuerde que puede ajustar los recursos en base a la carga. Por ejemplo, puede aumentar los recursos para crear un índice inicial completo y luego reducir los recursos más adelante a un nivel más adecuado para la indexación incremental.

> [!Important]
> Un servicio debe tener [2 réplicas para SLA de solo lectura y 3 réplicas para SLA de lectura y escritura](https://azure.microsoft.com/support/legal/sla/search/v1_0/).

1. Vaya a la página del servicio de búsqueda de Azure Portal.
2. En el panel de navegación de la izquierda, seleccione **Configuración** > **Escala**.
3. Use la barra deslizante para agregar recursos de cualquier tipo.

![Ampliación de capacidad](./media/search-create-service-portal/settings-scale.png "Aumento de capacidad mediante réplicas y particiones")

> [!Note]
> El almacenamiento y velocidad por partición aumenta en los niveles más altos. Para más información, consulte [capacidad y límites](search-limits-quotas-capacity.md).

## <a name="when-to-add-a-second-service"></a>Cuándo se debe agregar un segundo servicio

La mayoría de los clientes usan un solo servicio aprovisionado en un nivel que proporciona el [equilibrio de recursos adecuado](search-sku-tier.md). Un servicio puede hospedar varios índices en función de los [límites máximos de la capa que seleccione](search-capacity-planning.md), con cada índice aislado del otro. En Azure Cognitive Search, las solicitudes solo se podrán dirigir a un índice, lo que reduce al máximo la posibilidad de que se recuperen datos de forma voluntaria o involuntaria d otros índices en el mismo servicio.

Aunque la mayoría de los clientes usan un solo servicio, la redundancia de servicios puede ser necesaria si los requisitos operativos son los siguientes:

+ [Continuidad empresarial y recuperación ante desastres (BCDR)](https://docs.microsoft.com/azure/best-practices-availability-paired-regions). Azure Cognitive Search no proporciona conmutación por error instantánea si se produce una interrupción del servicio.

+ [Las arquitecturas multiinquilino](search-modeling-multitenant-saas-applications.md) a veces llaman a dos o más servicios.

+ Es posible que las aplicaciones implementadas globalmente requieran servicios de búsqueda en cada geografía para minimizar la latencia.

> [!NOTE]
> En Azure Cognitive Search, no es posible separar las operaciones de indexación y consulta; por consiguiente, no cree varios servicios para cargas de trabajo separadas. Un índice siempre se consulta en el servicio donde se creó (no se puede crear un índice en un servicio y copiar en otro).

No se requiere un segundo servicio para lograr alta disponibilidad. La alta disponibilidad en las consultas se logra al usar 2 o más réplicas en el mismo servicio. Las actualizaciones de réplicas son secuenciales, lo que significa que, al menos, una es operativa cuando se implementa una actualización de servicio. Para obtener más información, consulte [Acuerdos de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/search/v1_0/).

## <a name="next-steps"></a>Pasos siguientes

Después de aprovisionar un servicio, puede continuar en el portal para crear el primer índice.

> [!div class="nextstepaction"]
> [Inicio rápido: Creación de un índice de Azure Cognitive Search en el portal](search-get-started-portal.md)
