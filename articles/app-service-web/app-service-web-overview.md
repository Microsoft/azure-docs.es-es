---
title: "Introducción a Web Apps | Microsoft Docs"
description: "Vea cómo el Servicio de aplicaciones de Azure lo ayuda a desarrollar y hospedar aplicaciones web."
services: app-service\web
documentationcenter: 
author: cephalin
manager: erikre
editor: 
ms.assetid: 94af2caf-a2ec-4415-a097-f60694b860b3
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 01/04/2017
ms.author: cephalin
ms.custom: mvc
ms.translationtype: Human Translation
ms.sourcegitcommit: 6ea03adaabc1cd9e62aa91d4237481d8330704a1
ms.openlocfilehash: 2a1e1004a4dab48aed75740a24ff5556242eaa67
ms.contentlocale: es-es
ms.lasthandoff: 04/06/2017

---
# <a name="web-apps-overview"></a>Introducción a Aplicaciones web
*Aplicaciones web del Servicio de aplicaciones* es una plataforma de procesos completamente administrada que se ha optimizado para el hospedaje de sitios y aplicaciones web. La oferta de [plataforma como servicio](https://en.wikipedia.org/wiki/Platform_as_a_service) (PaaS) de Microsoft Azure lo permite centrarse en la lógica de negocios mientras Azure se encarga de la infraestructura para ejecutar y escalar las aplicaciones.

El siguiente vídeo de 5 minutos presenta Aplicaciones web del Servicio de aplicaciones de Azure.

>[!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Web-Apps-with-Yochay-Kiriaty/player]
>
>

> [!INCLUDE [app-service-linux](../../includes/app-service-linux.md)]
> 
> 

## <a name="what-is-a-web-app-in-app-service"></a>¿Qué es una aplicación web en el Servicio de aplicaciones?
En el Servicio de aplicaciones, una *aplicación web* son recursos de procesos que Azure proporciona para hospedar un sitio web o una aplicación web.  

Los recursos de procesos pueden ser máquinas virtuales compartidas o dedicadas, según el plan de tarifa que elija. El código de la aplicación se ejecuta en una máquina virtual administrada que se aísla de otros clientes.

El código puede utilizar cualquier lenguaje o marco que sea compatible con el [Servicio de aplicaciones de Azure](../app-service/app-service-value-prop-what-is.md), como ASP.NET, Node.js, Java, PHP o Python. También puede ejecutar scripts que usen [PowerShell y otros lenguajes de scripting](web-sites-create-web-jobs.md#acceptablefiles) en una aplicación web.

Para obtener ejemplos de escenarios de aplicación típicos en los que puede usar Web Apps, consulte [Escenarios de aplicaciones web](https://azure.microsoft.com/documentation/scenarios/web-app/) y la sección **Escenarios y recomendaciones** de [Comparación de Azure App Service, Virtual Machines, Service Fabric y Cloud Services](choose-web-site-cloud-service-vm.md#scenarios).

## <a name="why-use-web-apps"></a>¿Por qué usar Aplicaciones web?
Estas son algunas características clave del Servicio de aplicaciones que se aplican a Aplicaciones web:

* **Varios lenguajes y plataformas** : Servicio de aplicaciones es compatible con ASP.NET, Node.js, Java, PHP y Python. También puede ejecutar [PowerShell y otros scripts o ejecutables](web-sites-create-web-jobs.md) en máquinas virtuales del Servicio de aplicaciones.
* **Optimización de DevOps** : configure la [integración y la implementación continuas](app-service-continuous-deployment.md) con Visual Studio Team Services, GitHub o BitBucket. Promueva actualizaciones a través de [entornos de ensayo y prueba](web-sites-staged-publishing.md). Realice [las pruebas A/B](app-service-web-test-in-production-get-start.md). Administre las aplicaciones de App Service mediante [Azure PowerShell](/powershell/azureps-cmdlets-docs) o la [interfaz de la línea de comandos (CLI) multiplataforma](../cli-install-nodejs.md).
* **Escala global con alta disponibilidad**: escale [verticalmente](web-sites-scale.md) u [horizontalmente](../monitoring-and-diagnostics/insights-how-to-scale.md) de forma manual o automática. Hospede las aplicaciones en cualquier parte de la infraestructura del centro de datos global de Microsoft y el [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/app-service/) del Servicio de aplicaciones promete una alta disponibilidad.
* **Conexiones a plataformas SaaS y a datos locales** : elija entre más de 50 [conectores](../connectors/apis-list.md) para sistemas empresariales (como SAP, Siebel y Oracle), servicios de SaaS conocidos (como Salesforce y Office 365) y servicios de Internet (como Facebook y Twitter). Acceda a los datos locales mediante [Conexiones híbridas](../biztalk-services/integration-hybrid-connection-overview.md) y [Azure Virtual Networks](web-sites-integrate-with-vnet.md).
* **Seguridad y cumplimiento** : Servicio de aplicaciones cumple con [ISO, SOC y PCI](https://www.microsoft.com/TrustCenter/).
* **Plantillas de aplicación** : elija entre una amplia lista de plantillas de aplicación en [Azure Marketplace](https://azure.microsoft.com/marketplace/) que le permiten usar un asistente para instalar el software de código abierto popular, como WordPress, Joomla y Drupal.
* **Integración con visual Studio** : existen herramientas dedicadas en Visual Studio que permiten optimizar las tareas de creación, implementación y depuración.

Además, una aplicación web puede aprovechar las características ofrecidas por [API Apps](../app-service-api/app-service-api-apps-why-best-platform.md) (como la compatibilidad con CORS) y [Mobile Apps](../app-service-mobile/app-service-mobile-value-prop.md) (por ejemplo, las notificaciones push). Para más información sobre los tipos de aplicaciones del Servicio de aplicaciones, consulte [¿Qué es Servicios de aplicaciones de Azure?](../app-service/app-service-value-prop-what-is.md).

Además de Aplicaciones web en el Servicio de aplicaciones, Azure ofrece otros servicios que se pueden utilizar para hospedar aplicaciones web y sitios web. En la mayoría de los casos, Aplicaciones web es la mejor opción.  Para arquitectura de microservicios, puede usar [Service Fabric](https://azure.microsoft.com/documentation/services/service-fabric) y, si necesita más control sobre las máquinas virtuales en las que se ejecuta el código, piense utilizar [Azure Virtual Machines](https://azure.microsoft.com/documentation/services/virtual-machines/). Para más información sobre qué servicio de Azure elegir, consulte [Comparación de Servicio de aplicaciones de Azure, Servicios en la nube de Azure, Máquinas virtuales de Azure y Azure Service Fabric](choose-web-site-cloud-service-vm.md).

## <a name="getting-started"></a>Introducción
Para comenzar mediante la implementación de código de ejemplo en una nueva aplicación web en el App Service. A continuación, siga uno de los siguientes tutoriales del cuadro desplegable. Necesitará una cuenta gratis de Azure.

> [!div class="op_single_selector"]
> * [Implementación de la primer aplicación web ASP.NET en 5 minutos](app-service-web-get-started-dotnet.md)
> * [Implementación de la primer aplicación web PHP en Azure en 5 minutos](app-service-web-get-started-php.md)
> * [Implementación de la primer aplicación web Node.js en Azure en 5 minutos](app-service-web-get-started-nodejs.md)
> * [Implementación de la primer aplicación web Java en Azure en 5 minutos](app-service-web-get-started-java.md)
> * [Implementación de la primer aplicación web Python en Azure en 5 minutos](app-service-web-get-started-python.md)
> * [Implementación del primer sitio HTML en Azure en 5 minutos](app-service-web-get-started-html.md)
> 
> 

> [!NOTE]
> También puede [probar App Service](https://azure.microsoft.com/try/app-service/) sin una cuenta de Azure. Cree una aplicación de inicio y juegue con ella durante una hora como máximo; no se requiere ninguna tarjeta de crédito ni ningún compromiso.
> 
> 

