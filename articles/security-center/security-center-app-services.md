---
title: Protección de las aplicaciones web y API de Azure App Service
description: Este artículo le ayuda a comenzar a proteger las aplicaciones web y API de Azure App Services en Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: e8518710-fcf9-44a8-ae4b-8200dfcded1a
ms.service: security-center
ms.topic: conceptual
ms.date: 01/27/2019
ms.author: memildin
ms.openlocfilehash: 9fc5fac95d1fca743b8242f253d15be9a4b70853
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89277602"
---
# <a name="protect-your-azure-app-service-web-apps-and-apis"></a>Protección de las aplicaciones web y API de Azure App Service

Azure App Service es una plataforma totalmente administrada para crear y hospedar aplicaciones web y API sin preocuparse por tener que administrar la infraestructura. Proporciona administración, supervisión y conclusiones operativas para satisfacer requisitos de seguridad, cumplimiento normativo y rendimiento de nivel empresarial. Para más información, consulte [Azure App Service](https://azure.microsoft.com/services/app-service/).

Para habilitar la protección contra amenazas avanzada para su plan de Azure App Service, debe:

* Suscribirse a un plan de tarifa estándar de Azure Security Center
* Habilitar el plan de App Service tal como se muestra más adelante. Security Center está integrado de forma nativa con App Service, lo que elimina la necesidad de implementación e incorporación; la integración es transparente.
* Tener un plan de App Service asociado a máquinas dedicadas. Los planes admitidos son: Básico, Estándar, Premium, Aislado o Linux. Security Center no admite los planes Gratis, Compartido ni Consumo. Para más información, consulte [Planes de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/).

Con el plan de App Service habilitado, Security Center evalúa los recursos incluidos en el plan de App Service y genera recomendaciones de seguridad en función de sus hallazgos. Security Center protege la instancia de la VM en la que se está ejecutando la instancia de App Service y la interfaz de administración. También supervisa las solicitudes y respuestas enviadas a las aplicaciones que se ejecutan en App Service, y desde ellas.

Security Center aprovecha la escala de la nube y la visibilidad que tiene Azure como proveedor de servicios en la nube, para supervisar los ataques a las aplicaciones web comunes. Security Center puede descubrir ataques a las aplicaciones e identificar ataques emergentes, incluso mientras los atacantes se encuentran en la fase de reconocimiento, explorando para identificar vulnerabilidades en varias aplicaciones hospedadas en Azure. Como servicio nativo de Azure, Security Center también se encuentra en una posición única para ofrecer análisis de seguridad basados en host que cubren los nodos de proceso subyacentes para este PaaS, lo que permite a Security Center detectar ataques contra aplicaciones web que ya se habían aprovechado. Para más información, consulte [Protección contra amenazas para Azure App Service](threat-protection.md#app-services).


## <a name="enable-monitoring-and-protection-of-app-service"></a>Habilitación de la supervisión y la protección de App Service

1. En Azure Portal, elija Security Center.
2. Vaya a **Pricing & settings** (Precios y configuración) y elija una suscripción.
3. En **Plan de tarifa**, en la fila **App Service**, alterne el plan en **Habilitado**.

    [![Habilitación de App Services en su suscripción de nivel estándar](media/security-center-app-services/app-services-toggle.png)](media/security-center-app-services/app-services-toggle.png#lightbox)


>[!NOTE]
> El número de instancias enumeradas para la **cantidad de recursos** representa el número total de instancias de proceso, en todos los planes de App Service en esta suscripción, en ejecución en el momento en que ha abierto la hoja del plan de tarifa.
>
> Azure App Service ofrece una variedad de planes. El plan de App Service define el conjunto de recursos de proceso para que una aplicación web se ejecute. Son equivalentes a las granjas de servidores en el hospedaje web convencional. Pueden configurarse una o varias aplicaciones para que se ejecuten en los mismos recursos informáticos (o en el mismo plan de App Service).
>
>Para validar el recuento, vaya a "planes de App Service" en Azure Portal, donde puede ver el número de instancias de proceso que se usa en cada plan. 






Para deshabilitar la supervisión y recomendaciones para la instancia de App Service, repita este proceso y alterne el plan de **App Service** en **Deshabilitado**.



## <a name="see-also"></a>Consulte también
En este artículo, aprendió a usar las funcionalidades de supervisión en Azure Security Center. Para obtener más información sobre Azure Security Center, consulte los siguientes artículos:

* [Establecimiento de directivas de seguridad en Azure Security Center](tutorial-security-policy.md): Aprenda a configurar opciones de seguridad en Azure Security Center.
* [Administración y respuesta a las alertas de seguridad en Azure Security Center](security-center-managing-and-responding-alerts.md): Aprenda a administrar y responder a las alertas de seguridad.
* [App Services](security-center-virtual-machine-protection.md#app-services):  consulte una lista de los entornos de aplicaciones con resúmenes de estado.
* [Supervisión de las soluciones de asociados con Azure Security Center](security-center-partner-solutions.md): Aprenda cómo supervisar el estado de mantenimiento de las soluciones de sus asociados.
* [Blog de seguridad de Azure](https://blogs.msdn.com/b/azuresecurity/): Encuentre artículos de blog sobre el cumplimiento y la seguridad de Azure.
