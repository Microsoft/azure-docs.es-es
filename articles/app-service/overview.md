---
title: Información general
description: Vea cómo Azure App Service lo ayuda a desarrollar y hospedar aplicaciones web.
ms.assetid: 94af2caf-a2ec-4415-a097-f60694b860b3
ms.topic: overview
ms.date: 07/06/2020
ms.custom: devx-track-dotnet, mvc, seodec18
ms.openlocfilehash: 1b72224441741990a1fc94400dfe718ea9d1b0b3
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961489"
---
# <a name="app-service-overview"></a>Información general de App Service

*Azure App Service* es un servicio basado en HTTP para hospedar aplicaciones web, API REST y back-ends para dispositivos móviles. Puede desarrollarlo en su lenguaje preferido, ya sea. NET, .NET Core, Java, Ruby, Node.js, PHP o Python. Las aplicaciones se ejecutan y escalan fácilmente en los entornos basados tanto en Windows como en [Linux](#app-service-on-linux).

App Service no solo agrega a la aplicación la funcionalidad de Microsoft Azure, como la seguridad, el equilibrio de carga, el escalado automático y la administración automatizada. También puede sacar partido de sus funcionalidades de DevOps, por ejemplo, la implementación continua desde Azure DevOps, GitHub, Docker Hub y otros orígenes, la administración de paquetes, entornos de ensayo, dominio personalizado y certificados TLS/SSL. 

Con App Service, se paga por los recursos de proceso de Azure que se utilizan. Los recursos de proceso que usa se determinan mediante el _plan de App Service_ en el que ejecuta las aplicaciones. Para más información, consulte [Información general sobre los planes de Azure App Service](overview-hosting-plans.md).

## <a name="why-use-app-service"></a>¿Por qué usar App Service?

Estas son algunas características clave de App Service:

* **Varios lenguajes y plataformas**: App Service tiene compatibilidad de primera clase con ASP.NET, ASP.NET Core, Java, Ruby, Node.js, PHP o Python. También puede ejecutar [PowerShell y otros scripts o ejecutables](webjobs-create.md) como servicios en segundo plano.
* **Entorno de producción administrado**: App Service [parchea y mantiene los marcos del sistema operativo y del lenguaje](overview-patch-os-runtime.md) de forma automática. Invierta su tiempo en escribir aplicaciones magníficas y deje que Azure se preocupe por la plataforma.
* **Contenedores y Docker**: aplique Docker a la aplicación y hospede un contenedor de Windows o Linux personalizado en App Service. Ejecute las aplicaciones de varios contenedores con Docker Compose. Migre sus habilidades de Docker directamente a App Service.
* **Optimización con DevOps**: configure la [integración y la implementación continuas](deploy-continuous-deployment.md) con Azure DevOps, GitHub, BitBucket, Docker Hub o Azure Container Registry. Promueva actualizaciones a través de [entornos de ensayo y prueba](deploy-staging-slots.md). Administre las aplicaciones de App Service mediante [Azure PowerShell](/powershell/azure/) o la [interfaz de la línea de comandos (CLI) multiplataforma](/cli/azure/install-azure-cli).
* **Escala global con alta disponibilidad**: escale [verticalmente](manage-scale-up.md) u [horizontalmente](../azure-monitor/platform/autoscale-get-started.md) de forma manual o automática. Hospede las aplicaciones en cualquier parte de la infraestructura del centro de datos global de Microsoft y el [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/app-service/) de App Service promete una alta disponibilidad.
* **Conexiones a plataformas SaaS y a datos locales**: elija entre más de 50 [conectores](../connectors/apis-list.md) para sistemas empresariales (como SAP), servicios SaaS (como Salesforce) y servicios de Internet (como Facebook). Acceda a los datos locales mediante [Conexiones híbridas](app-service-hybrid-connections.md) y [Azure Virtual Networks](web-sites-integrate-with-vnet.md).
* **Seguridad y cumplimiento**: App Service cumple con [ISO, SOC y PCI](https://www.microsoft.com/en-us/trustcenter). Autentique a los usuarios con [Azure Active Directory](configure-authentication-provider-aad.md), [Google](configure-authentication-provider-google.md), [Facebook](configure-authentication-provider-facebook.md), [Twitter](configure-authentication-provider-twitter.md) o una [cuenta Microsoft](configure-authentication-provider-microsoft.md). Cree [restricciones de direcciones IP](app-service-ip-restrictions.md) y [administre las identidades de servicio](overview-managed-identity.md).
* **Plantillas de aplicación**: elija entre una amplia lista de plantillas de aplicación en [Azure Marketplace](https://azure.microsoft.com/marketplace/), como WordPress, Joomla y Drupal.
* **Integración con Visual Studio y Visual Studio Code** : existen herramientas dedicadas en Visual Studio y Visual Studio Code que permiten optimizar las tareas de creación, implementación y depuración.
* **API y características para móviles**: App Service proporciona compatibilidad CORS llave en mano para escenarios de la API RESTful y simplifica los escenarios de aplicaciones móviles al permitir la autenticación, la sincronización de datos sin conexión, las notificaciones push, y mucho más.
* **Código sin servidor**: ejecute un fragmento de código o script a petición sin tener que proporcionar explícitamente ni administrar la infraestructura, y pague solo por el tiempo de proceso que el código utiliza realmente (vea [Azure Functions](../azure-functions/index.yml)).

Además de App Service, Azure ofrece otros servicios que se pueden usar para hospedar aplicaciones web y sitios web. Para la mayoría de los escenarios, App Service es la mejor opción.  En el caso de la arquitectura de microservicios, considere la posibilidad de usar [Azure Spring-Cloud Service](../spring-cloud/index.yml) o [Service Fabric](https://azure.microsoft.com/documentation/services/service-fabric).  Si necesita más control sobre las máquinas virtuales en las que se ejecuta el código, considere la posibilidad de utilizar [Azure Virtual Machines](https://azure.microsoft.com/documentation/services/virtual-machines/). Para más información sobre qué servicio de Azure elegir, consulte [Comparación de Azure App Service, Virtual Machines, Service Fabric y Cloud Services](/azure/architecture/guide/technology-choices/compute-decision-tree).

## <a name="app-service-on-linux"></a>App Service en Linux

App Service también puede hospedar las aplicaciones Web de forma nativa en Linux para las pilas de aplicaciones admitidas. Además, puede ejecutar contenedores de Linux personalizados (también conocidos como Web App for Containers).

### <a name="built-in-languages-and-frameworks"></a>Lenguajes y marcos integrados

App Service en Linux admite varias imágenes integradas específicas del lenguaje. Solo implemente el código. Los lenguajes compatibles incluyen: Node.js, Java (JRE 8 y JRE 11), PHP, Python, .NET Core y Ruby. Ejecute [`az webapp list-runtimes --linux`](/cli/azure/webapp?view=azure-cli-latest#az-webapp-list-runtimes) para ver los idiomas y las versiones admitidas más recientes. Si el motor de tiempo de ejecución que requiere la aplicación no se admite en las imágenes integradas, puede implementarlo con un contenedor personalizado.

### <a name="limitations"></a>Limitaciones

- App Service en Linux no se admite en el plan de tarifa [Compartido](https://azure.microsoft.com/pricing/details/app-service/plans/). 
- No se pueden mezclar las aplicaciones Windows y Linux en el mismo plan de App Service.  
- Dentro del mismo grupo de recursos, no se pueden mezclar aplicaciones Windows y Linux en la misma región.
- Azure Portal solo muestra las características que funcionan actualmente para las aplicaciones Linux. A medida que se habiliten las características, se activarán en el portal.
- Cuando se implementen en imágenes integradas, el código y el contenido se asignarán a un volumen de almacenamiento para el contenido web, respaldado por Azure Storage. La latencia de disco de este volumen es mayor y más variable que la del sistema de archivos del contenedor. Las aplicaciones que requieran muchos accesos de solo lectura a archivos de contenido pueden beneficiarse de la implementación de contenedores personalizados, que permite colocar los archivos en el sistema de archivos de contenedor en lugar de en el volumen de contenido.

## <a name="next-steps"></a>Pasos siguientes

Cree su primera aplicación web.

> [!div class="nextstepaction"]
> [ASP.NET Core (en Windows o Linux)](quickstart-dotnetcore.md)

> [!div class="nextstepaction"]
> [ASP.NET (en Windows)](quickstart-dotnet-framework.md)

> [!div class="nextstepaction"]
> [PHP (en Windows o Linux)](quickstart-php.md)

> [!div class="nextstepaction"]
> [Ruby (en Linux)](quickstart-ruby.md)

> [!div class="nextstepaction"]
> [Node.js (en Windows o Linux)](quickstart-nodejs.md)

> [!div class="nextstepaction"]
> [Java (en Windows o Linux)](quickstart-java.md)

> [!div class="nextstepaction"]
> [Python (en Linux)](quickstart-python.md)

> [!div class="nextstepaction"]
> [HTML (en Windows o Linux)](quickstart-html.md)

> [!div class="nextstepaction"]
> [Contenedor personalizado (Windows o Linux)](tutorial-custom-container.md)