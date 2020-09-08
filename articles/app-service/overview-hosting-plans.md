---
title: Planes de App Service
description: Obtenga información sobre cómo funcionan los planes de App Service en Azure App Service, cómo se facturan al cliente y cómo escalarlos según sus necesidades.
keywords: app service, azure app service, escala, escalable, escalabilidad, plan de app service, costo de app service
ms.assetid: dea3f41e-cf35-481b-a6bc-33d7fc9d01b1
ms.topic: article
ms.date: 08/12/2020
ms.custom: seodec18
ms.openlocfilehash: f30221de81b6bef199c0a25e770558c4db8c4006
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958519"
---
# <a name="azure-app-service-plan-overview"></a>Introducción a los planes de Azure App Service

En App Service (Web Apps, API Apps o Mobile Apps), una aplicación siempre se ejecuta en un _plan de App Service_. Además, [Azure Functions](../azure-functions/functions-scale.md#app-service-plan) se puede ejecutar en un _plan de App Service_. Un plan de App Service define un conjunto de recursos de proceso para que una aplicación web se ejecute. Estos recursos de proceso son análogos a la [_granja de servidores_](https://wikipedia.org/wiki/Server_farm) de un hospedaje web convencional. Pueden configurarse una o varias aplicaciones para que se ejecuten en los mismos recursos informáticos (o en el mismo plan de App Service).

Cuando se crea un plan de App Service en una región determinada (por ejemplo, Oeste de Europa), se crea un conjunto de recursos de proceso para ese plan en dicha región. Todas las aplicaciones que coloque en este plan de App Service se ejecutan en estos recursos de proceso según lo definido por el plan de App Service. Cada plan de App Service define:

- Región (oeste de EE. UU., este de EE. UU., etc.)
- Número de instancias de VM
- Tamaño de las instancias de VM (pequeño, mediano, grande)
- Plan de tarifa (Gratis, Compartido, Básico, Estándar, Premium, PremiumV2 y Aislado)

El _plan de tarifa_ de un plan de App Service determina qué características de App Service obtendrá y cuánto paga por el plan. Existen algunas categorías de planes de tarifa:

- **Proceso de compartido**: **Gratis** y **Compartido**, los dos planes básicos, ejecutan una aplicación en la misma VM de Azure que otras aplicaciones de App Service, incluidas las aplicaciones de otros clientes. Estos planes asignan cuotas de CPU a cada aplicación que se ejecuta en los recursos compartidos, y los recursos no pueden escalarse horizontalmente.
- **Dedicated compute** (Proceso dedicado): Los planes **Básico**, **Estándar**, **Premium** y **PremiumV2** ejecutan aplicaciones en VM de Azure dedicadas. Solo las aplicaciones del mismo plan de App Service comparten los mismos recursos de proceso. Cuanto mayor sea el plan, más instancias de VM estarán disponibles para la escalabilidad horizontal.
- **Aislado**: Este nivel ejecuta máquinas virtuales de Azure dedicadas en instancias de Microsoft Azure Virtual Network. Proporciona aislamiento de red, además de aislamiento de proceso a sus aplicaciones. Proporciona las máximas posibilidades de escalabilidad horizontal.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

Además, cada plan proporciona un subconjunto específico de características de App Service. Estas características incluyen dominios personalizados y certificados TLS/SSL, escalado automático, ranuras de implementación, copias de seguridad, integración de Traffic Manager y mucho más. Cuanto mayor sea el plan, más características estarán disponibles. Para averiguar qué características se admiten en cada plan de tarifa, consulte los [detalles del plan de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/).

<a name="new-pricing-tier-premiumv2"></a>

> [!NOTE]
> El nuevo plan de tarifa **PremiumV2** incluye las [máquinas virtuales de la serie Dv2](../virtual-machines/dv2-dsv2-series.md), que cuentan con procesadores más rápidos, almacenamiento SSD y el doble de memoria en proporción de núcleo en comparación con el nivel **Estándar**. **PremiumV2** permite también una escala mayor con un número más alto de instancias, al tiempo que proporciona toda la funcionalidad avanzada del plan Estándar. Todas las características disponibles en el nivel **Premium** existente se incluyen en el nivel **PremiumV2**.
>
> De manera similar a otros niveles dedicados, existen tres tamaños de VM disponibles para este nivel:
>
> - Pequeño (un núcleo de CPU, 3,5 GiB de memoria) 
> - Mediano (dos núcleos de CPU, 7 GiB de memoria) 
> - Grande (cuatro núcleos de CPU, 14 GiB de memoria)  
>
> Para obtener más información sobre el precio del plan **PremiumV2**, consulte [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/).
>
> Para empezar a usar el nuevo plan de tarifa **PremiumV2**, consulte [Configuración del nivel PremiumV2 para App Service](app-service-configure-premium-tier.md).

## <a name="how-does-my-app-run-and-scale"></a>¿Cómo se ejecuta y escala mi aplicación?

En los planes **Gratis** y **Compartido**, una aplicación recibe minutos de CPU en una instancia compartida de VM y no se puede escalar horizontalmente. En otros planes, una aplicación se ejecuta y escala como se indica a continuación.

Cuando crea una aplicación en App Service, se coloca en un plan de App Service. Cuando se ejecuta la aplicación, se ejecuta en todas las instancias de VM configuradas en el plan de App Service. Si hay varias aplicaciones en el mismo plan de App Service, comparten las mismas instancias de VM. Si tiene varias ranuras de implementación para una aplicación, todas las ranuras de implementación se ejecutan también en las mismas instancias de VM. Si habilita los registros de diagnóstico, realiza copias de seguridad o ejecuta WebJobs, también usan ciclos de CPU y memoria en estas instancias de VM.

De esta manera, el plan de App Service es la unidad de escalado de las aplicaciones de App Service. Si el plan está configurado para ejecutar cinco instancias de VM, todas las aplicaciones del plan se ejecutan en las cinco instancias. Si el plan está configurado para el escalado automático, todas las aplicaciones del plan se escalan horizontalmente juntas según la configuración de escalado automático.

Para obtener información sobre el escalado horizontal de una aplicación, consulte [Escalado del recuento de instancias de forma manual o automática](../azure-monitor/platform/autoscale-get-started.md).

<a name="cost"></a>

## <a name="how-much-does-my-app-service-plan-cost"></a>¿Cuánto cuesta mi plan de App Service?

En esta sección se describe cómo se facturan las aplicaciones de App Service. Para obtener información detallada sobre precios para regiones específicas, consulte [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/).

Salvo en el caso del nivel **Gratis**, un plan de App Service conlleva un cargo asociado a los recursos de proceso que se utilicen.

- En el nivel **Compartido**, cada aplicación recibe una cuota de minutos de CPU, por lo que _cada aplicación_ tiene un cargo asociado a la cuota de CPU.
- En los niveles de proceso dedicados (**Básico**, **Estándar**, **Premium** y **PremiumV2**), el plan de App Service define el número de instancias de máquina virtual al que se escalan las aplicaciones, de manera que se apliquen cargos a _cada instancia de máquina virtual_ del plan de App Service. Estas instancias de VM se cobran igual, independientemente de cuántas aplicaciones se ejecuten en ellas. Para evitar cargos inesperados, consulte [Clean up an App Service plan](app-service-plan-manage.md#delete) (Eliminar un plan de App Service).
- En el nivel **Aislado**, el entorno de App Service define el número de trabajos aislados que ejecutan las aplicaciones, y se aplican cargos a _cada trabajo_. Además, hay una tarifa plana para el stamp por la ejecución de App Service Environment.

No se le cobra por usar las características de App Service que tiene a su disposición (configurar dominios personalizados, certificados TLS/SSL, ranuras de implementación, copias de seguridad, etc.). Las excepciones son estas:

- Dominios de App Service: paga al adquirir uno en Azure y cuando lo renueva cada año.
- App Service Certificate: paga al adquirir uno en Azure y cuando lo renueva cada año.
- Conexiones TLS basadas en IP: existe un cargo por hora para cada conexión TLS basada en IP, pero algunos planes**Estándar** o superiores ofrecen una conexión TLS basada en IP de forma gratuita. Las conexiones TLS basadas en SNI son gratuitas.

> [!NOTE]
> Si integra App Service con otro servicio de Azure, debe tener en cuenta los cargos de estos otros servicios. Por ejemplo, si utiliza Azure Traffic Manager para escalar la aplicación geográficamente, Azure Traffic Manager también le cobra en función del uso. Para calcular el costo entre servicios de Azure, consulte [Calculadora de precios](https://azure.microsoft.com/pricing/calculator/). 

¿Quiere optimizar y ahorrar en el gasto en la nube?

[!INCLUDE [cost-management-horizontal](../../includes/cost-management-horizontal.md)]

## <a name="what-if-my-app-needs-more-capabilities-or-features"></a>¿Qué ocurre si mi aplicación necesita más funcionalidades o características?

El plan de App Service se puede escalar o reducir verticalmente en cualquier momento. Basta con cambiar el plan de tarifa del plan. Puede elegir un plan de tarifa inferior al principio y escalar verticalmente más adelante cuando necesite más características de App Service.

Por ejemplo, puede comenzar a probar una aplicación web en un plan de App Service **Gratis** y no pagar nada. Cuando quiera agregar su [nombre DNS personalizado](app-service-web-tutorial-custom-domain.md) a la aplicación web, simplemente, escale el plan verticalmente hasta un plan **Compartido**. Más adelante, cuando desee [crear un enlace TLS](configure-ssl-bindings.md), escale el plan hasta el nivel **Básico**. Si desea tener [entornos de ensayo](deploy-staging-slots.md), escale verticalmente hasta el plan **Estándar**. Cuando necesite más núcleos, memoria o almacenamiento, escale verticalmente a un tamaño superior de VM del mismo nivel.

Funciona igual a la inversa. Cuando crea que ya no necesita las funcionalidades o características de un plan superior, puede reducir verticalmente a un plan inferior, lo que permite ahorrar dinero.

Para obtener más información sobre el escalado vertical del plan de App Service, consulte [Escalado vertical de aplicaciones en Azure](manage-scale-up.md).

Si la aplicación está en el mismo plan de App Service con otras aplicaciones, puede que desee mejorar el rendimiento de la aplicación aislando los recursos de proceso. Para hacerlo, puede mover la aplicación a otro plan de App Service. Para obtener más información, consulte [Move an app to another App Service plan](app-service-plan-manage.md#move) (Mover una aplicación a otro plan de App Service).

## <a name="should-i-put-an-app-in-a-new-plan-or-an-existing-plan"></a>¿Debería colocar una aplicación en un nuevo plan o en un plan existente?

Puesto que paga por los recursos informáticos que asigna su plan de App Service (consulte [¿Cuánto cuesta mi plan de App Service?](#cost)), posiblemente pueda ahorrar dinero al colocar varias aplicaciones en un plan de App Service. Puede seguir agregando aplicaciones a un plan existente siempre que el plan tenga suficientes recursos para administrar la carga. No obstante, tenga en cuenta que las aplicaciones del mismo plan de App Service comparten los mismos recursos de proceso. Para determinar si la nueva aplicación tiene los recursos necesarios, debe comprender la capacidad del plan de App Service existente y la carga prevista para la nueva aplicación. La sobrecarga de un plan de App Service puede provocar tiempos de inactividad en aplicaciones nuevas y existentes.

Aísle la aplicación en un nuevo plan de App Service en los siguientes casos:

- La aplicación consume muchos recursos.
- Quiere escalar la aplicación independientemente de las demás aplicaciones del plan existente.
- La aplicación necesita recursos de una región geográfica diferente.

De esta forma, puede asignar un nuevo conjunto de recursos para la aplicación y tener un mayor control de las aplicaciones.

## <a name="manage-an-app-service-plan"></a>Administración de un plan de App Service

> [!div class="nextstepaction"]
> [Administración de un plan de App Service](app-service-plan-manage.md)