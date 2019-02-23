---
title: Guía de introducción para desarrolladores en Azure | Microsoft Docs
description: En este tema se proporciona información esencial para los desarrolladores que desean comenzar a usar la plataforma Microsoft Azure para sus necesidades de desarrollo.
services: ''
cloud: ''
documentationcenter: ''
author: ggailey777
manager: erikre
ms.assetid: ''
ms.service: na
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/18/2017
ms.author: glenga
ms.openlocfilehash: 7c4eefa09f0a70d42601a5b1fe8694500b5c6be2
ms.sourcegitcommit: 818d3e89821d101406c3fe68e0e6efa8907072e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/09/2019
ms.locfileid: "54118258"
---
# <a name="get-started-guide-for-azure-developers"></a>Guía de introducción para desarrolladores de Azure

## <a name="what-is-azure"></a>¿Qué es Azure?

Azure es una plataforma de nube completa que puede hospedar sus aplicaciones existentes, simplificar el desarrollo de nuevas aplicaciones e incluso mejorar las aplicaciones locales. Azure integra los servicios en la nube que necesita para desarrollar, probar, implementar y administrar sus aplicaciones, mientras aprovecha las ventajas de la informática en la nube.

Con el hospedaje de las aplicaciones en Azure, puede empezar con tamaño pequeño y escalar fácilmente su aplicación a medida que aumente la demanda de los clientes. Azure también ofrece la confiabilidad que se necesita para las aplicaciones de alta disponibilidad, incluso para la conmutación por error entre diferentes regiones. [Azure Portal](https://portal.azure.com) le permite administrar fácilmente todos los servicios de Azure. También puede administrar los servicios mediante programación, con las API y las plantillas específicas del servicio.

**¿A quién va dirigida esta información?**: Esta guía es una introducción a la plataforma Azure para desarrolladores de aplicaciones. Proporciona instrucciones y orientación para iniciar la creación de nuevas aplicaciones en Azure, o para migrar aplicaciones existentes a Azure.

## <a name="where-do-i-start"></a>¿Por dónde empiezo?

Con todos los servicios que Azure ofrece, averiguar qué servicios necesita para su arquitectura de soluciones puede resultar abrumador. En esta sección se resaltan los servicios de Azure que normalmente utilizan los desarrolladores. Para obtener una lista de todos los servicios de Azure, consulte la [documentación de Azure](../../index.md).

En primer lugar, debe decidir cómo hospedará su aplicación en Azure. ¿Necesita administrar toda la infraestructura como una máquina virtual (VM)? ¿Puede usar las funciones de administración de plataforma que proporciona Azure? ¿Tal vez necesite un entorno sin servidor para ejecutar código host únicamente?

Su aplicación necesita almacenamiento en la nube, para lo cual Azure ofrece varias opciones. Puede aprovechar las ventajas de la autenticación empresarial de Azure. También hay herramientas de desarrollo y supervisión basadas en la nube, y la mayoría de servicios de hospedaje ofrece integración con DevOps.

Ahora, echemos un vistazo a algunos de los servicios específicos que recomendamos investigar para sus aplicaciones.

### <a name="application-hosting"></a>Hospedaje de aplicaciones

Azure tiene varias ofertas de proceso en la nube para ejecutar su aplicación y que no tenga que preocuparse sobre los detalles de la infraestructura de proceso. Puede escalar fácilmente los recursos vertical u horizontalmente a medida que aumente el uso de la aplicación.

Azure ofrece servicios para sus necesidades de desarrollo y hospedaje de aplicaciones. Azure proporciona infraestructura como servicio (IaaS) para darle un control total sobre el hospedaje de las aplicaciones. Las ofertas de plataforma como servicio (PaaS) de Azure proporcionan los servicios completamente administrados que sus aplicaciones necesitan. Existe incluso un hospedaje sin servidor en Azure, en el que todo lo que necesita hacer es escribir su código.

![Opciones de hospedaje de aplicaciones en Azure](./media/azure-developer-guide/azure-developer-hosting-options.png)


#### <a name="azure-app-service"></a>Azure App Service 

Si desea el camino más corto para publicar proyectos web, considere Azure App Service. Con App Service es muy fácil extender las aplicaciones web para admitir a los clientes móviles y publicar sencillas API de REST. Esta plataforma proporciona autenticación mediante el uso de proveedores de redes sociales, autoescala basada en el tráfico, pruebas en producción e implementaciones de contenedor y continuas.

Puede crear aplicaciones web, back-ends de aplicaciones móviles y aplicaciones de API.

Como los tres tipos de aplicación comparten el entorno de tiempo de ejecución de App Service, puede hospedar un sitio web, admitir clientes móviles y exponer sus API en Azure, todo ello desde el mismo proyecto o solución. Para más información sobre App Service, vea [¿Qué es Azure Web Apps?](../../app-service/overview.md)

App Service se ha diseñado teniendo en cuenta DevOps. Admite varias herramientas de publicación e implementaciones de integración continuas, incluidos webhooks de GitHub, Jenkins, Azure DevOps, TeamCity y otros.

Puede migrar las aplicaciones existentes a App Service con la [herramienta de migración en línea](https://www.migratetoazure.net/).

>**Cuándo se debe usar**: use App Service al migrar aplicaciones web existentes a Azure y cuando necesite una plataforma de hospedaje completamente administrada para sus aplicaciones web. También puede utilizar App Service cuando necesite admitir clientes móviles o exponer API de REST con su aplicación.

>**Para comenzar**: con App Service resulta muy sencillo crear e implementar su primera [aplicación web](../../app-service/app-service-web-get-started-dotnet.md), [aplicación móvil](../../app-service-mobile/app-service-mobile-ios-get-started.md) o [aplicación de API](../../app-service/app-service-web-tutorial-rest-api.md).

>**Pruébelo ya**: App Service le permite aprovisionar una aplicación de corta duración para probar la plataforma sin necesidad de registrarse para obtener una cuenta de Azure. Pruebe la plataforma y [cree una aplicación de Azure App Service](https://tryappservice.azure.com/).

#### <a name="azure-virtual-machines"></a>Azure Virtual Machines

Como proveedor de infraestructura como servicio (IaaS), Azure le permite implementar o migrar la aplicación a máquinas virtuales Windows o Linux. Junto con Azure Virtual Network, Azure Virtual Machines permite la implementación de máquinas virtuales Windows o Linux en Azure. Con las máquinas virtuales, tiene un control total sobre la configuración de la máquina. Al usar las máquinas virtuales, es responsabilidad suya la instalación, la configuración y el mantenimiento del software del servidor, así como las revisiones del sistema operativo.

El nivel de control que tiene con las máquinas virtuales le permite ejecutar una amplia variedad de cargas de trabajo de servidor en Azure que no se ajustan a un modelo de PaaS. Estas cargas de trabajo incluyen servidores de base de datos, Windows Server Active Directory y Microsoft SharePoint. Para más información, consulte la documentación de Virtual Machines para [Linux](/azure/virtual-machines/linux/) o [Windows](/azure/virtual-machines/windows/).

>**Cuándo se debe usar**: use Virtual Machines cuando desee un control total de la infraestructura de su aplicación o para migrar las cargas de trabajo de aplicaciones locales a Azure sin tener que hacer cambios.

>**Para comenzar**: cree una [máquina virtual Linux](../../virtual-machines/virtual-machines-linux-quick-create-portal.md) o una [máquina virtual Windows](../../virtual-machines/virtual-machines-windows-hero-tutorial.md) en Azure Portal.

#### <a name="azure-functions-serverless"></a>Azure Functions (sin servidor)

En lugar de tener que preocuparse de crear y administrar una aplicación completa o la infraestructura para ejecutar el código, ¿qué le parecería poder escribir el código y hacer que se ejecute en respuesta a eventos o según una programación?  [Azure Functions](../../azure-functions/functions-overview.md) es una oferta "sin servidor" que le permite escribir solo el código que necesita. Con Functions, la ejecución del código se desencadena mediante solicitudes HTTP, webhooks, eventos de servicios en la nube o según una programación. El desarrollo se puede realizar en el lenguaje que se prefiera, como C\#, F\#, Node.js, Python o PHP. Con la facturación basada en el consumo, solo paga por el tiempo que el código se ejecuta y Azure escala según sea necesario.

>**Cuándo se debe usar**: use Azure Functions si tiene código que se desencadena mediante otros servicios de Azure, por medio de eventos basados en web o según una programación. También puede usar Functions cuando no necesite la sobrecarga de un proyecto completo hospedado o si desea pagar solo por el tiempo que el código se ejecuta. Para más información, consulte [Introducción a Azure Functions](../../azure-functions/functions-overview.md).

>**Para comenzar**: siga el tutorial de inicio rápido de Functions para [crear la primera función](../../azure-functions/functions-create-first-azure-function.md) desde el portal.

>**Pruébelo ya**: Azure Functions permite ejecutar código sin necesidad de registrarse para obtener una cuenta de Azure. Pruébelo ahora y [cree su primera función de Azure](https://tryappservice.azure.com/).

#### <a name="azure-service-fabric"></a>Azure Service Fabric

Azure Service Fabric es una plataforma de sistemas distribuidos que facilita la compilación, el empaquetamiento, la implementación y la administración de microservicios escalables y confiables. También proporciona capacidades de administración de aplicaciones completas para el aprovisionamiento, la implementación, la supervisión, la actualización/aplicación de revisiones y eliminación de aplicaciones implementadas. Las aplicaciones, que se ejecutan en un grupo compartido de máquinas, pueden empezar con pocas máquinas y escalar hasta cientos o miles de máquinas si es necesario.

Service Fabric admite WebAPI con Open Web Interface para .NET (OWIN) y ASP.NET Core. Ofrece varios SDK para compilar servicios en Linux tanto en .NET Core como Java. Para obtener más información acerca de Service Fabric, consulte la [documentación de Service Fabric](https://docs.microsoft.com/azure/service-fabric/).

>**Cuándo se debe usar:** Service Fabric es una buena opción si se va a crear una aplicación o se quiere volver a escribir una aplicación existente para usar una arquitectura de microservicios. Use Service Fabric cuando necesite más control sobre la infraestructura subyacente, o acceso directo a ella.

>**Introducción:** [Creación de la primera aplicación de Azure Service Fabric](../../service-fabric/service-fabric-create-your-first-application-in-visual-studio.md).

### <a name="enhance-your-applications-with-azure-services"></a>Mejore sus aplicaciones con los servicios de Azure

Además de hospedar aplicaciones, Azure ofrece servicios que pueden mejorar la funcionalidad, el desarrollo y el mantenimiento de las aplicaciones, tanto en la nube como en instalaciones locales.

#### <a name="hosted-storage-and-data-access"></a>Acceso de datos y almacenamiento hospedado

La mayoría de las aplicaciones debe almacenar datos; por lo tanto, independientemente de cómo decida hospedar la aplicación en Azure, considere la posibilidad de usar uno o varios de los siguientes servicios de datos y almacenamiento.

-   **Azure Cosmos DB**: un servicio de base de datos multimodelo de distribución global que le permite escalar de forma flexible el rendimiento y el almacenamiento entre cualquier cantidad de regiones geográficas con un Acuerdo de Nivel de Servicio completo. 
    >**Cuándo se debe usar:** cuando la aplicación necesite bases de datos de grafos, tablas o documentos, incluidas las bases de datos de MongoDB, con varios modelos de coherencia bien definidos. 

    >**Para comenzar**: [cree una aplicación web de Azure Cosmos DB](../../cosmos-db/create-sql-api-dotnet.md). Si es desarrollador de MongoDB, consulte [Azure Cosmos DB: Compilar una aplicación web de API MongoDB con .NET y Azure Portal](../../cosmos-db/create-mongodb-dotnet.md).

-   **Azure Storage**: ofrece almacenamiento duradero y de alta disponibilidad para blobs, colas, archivos y otros tipos de datos no relacionales. Storage proporciona la base de almacenamiento para las máquinas virtuales.

    >**Cuándo se debe usar**: cuando la aplicación almacene datos no relacionales, como pares de clave-valor (tablas), blobs, recursos compartidos de archivos o mensajes (colas).

    >**Para comenzar**: elija uno de estos tipos de almacenamiento: [blobs](../../storage/blobs/storage-dotnet-how-to-use-blobs.md), [tablas](../../cosmos-db/table-storage-how-to-use-dotnet.md), [colas](../../storage/queues/storage-dotnet-how-to-use-queues.md) o [archivos](../../storage/files/storage-dotnet-how-to-use-files.md).

-   **Azure SQL Database**: versión para Azure del motor de Microsoft SQL Server para almacenar datos tabulares relacionales en la nube. SQL Database ofrece un rendimiento predecible, escalabilidad sin tiempo de inactividad, continuidad empresarial y protección de datos.

    >**Cuándo se debe usar**: cuando la aplicación requiera almacenamiento de datos con integridad referencial, compatibilidad transaccional y compatibilidad con consultas TSQL.

    >**Para comenzar**: [cree una base de datos SQL en cuestión de minutos con Azure Portal](../../sql-database/sql-database-get-started.md).


Puede usar [Azure Data Factory](../../data-factory/introduction.md) para mover los datos locales existentes a Azure. Si no está listo para mover los datos a la nube, [Conexiones híbridas](../../biztalk-services/integration-hybrid-connection-overview.md) en BizTalk Services le permite conectar su aplicación hospedada en App Service con recursos locales. También puede conectarse a los servicios de datos y almacenamiento de Azure desde las aplicaciones locales.

#### <a name="docker-support"></a>Compatibilidad con Docker

Los contenedores de Docker, una forma de virtualización del sistema operativo, le permiten implementar aplicaciones de forma más eficaz y predecible. Una aplicación en contenedores funciona en producción de la misma manera que en los sistemas de desarrollo y pruebas. Puede administrar los contenedores mediante las herramientas estándar de Docker. Puede utilizar sus conocimientos y las herramientas de código abierto más populares para implementar y administrar en Azure aplicaciones basadas en contenedores.

Azure proporciona varias maneras de utilizar contenedores en sus aplicaciones.

-   **Extensión de máquina virtual de Docker**: permite configurar una máquina virtual con herramientas de Docker para que actúe como un host de Docker.

    >**Cuándo se debe usar**: cuando quiera generar implementaciones de contenedor coherentes para las aplicaciones en una máquina virtual, o cuando desee usar [Docker Compose](https://docs.docker.com/compose/overview/).

    >**Para comenzar**: [Cree un entorno de Docker en Azure mediante la extensión de máquina virtual de Docker](../../virtual-machines/virtual-machines-linux-dockerextension.md).

-   **Azure Container Service**: permite crear, configurar y administrar un clúster de máquinas virtuales preconfiguradas para ejecutar aplicaciones en contenedor. Para más información sobre Container Service, consulte [Introducción a Azure Container Service](../../container-service/container-service-intro.md).

    >**Cuándo se debe usar**: cuando necesite crear entornos escalables listos para producción que proporcionen herramientas de administración y programación adicionales, o para implementar un clúster de Docker Swarm.

    >**Para comenzar**: [implemente un clúster de Container Service](../../container-service/dcos-swarm/container-service-deployment.md).

-   **Máquina de Docker**: permite instalar y administrar un motor de Docker en hosts virtuales mediante comandos docker-machine.

    >**Cuándo se debe usar**: cuando sea necesario crear rápidamente un prototipo de una aplicación mediante la creación de un único host de Docker.

-   **Imagen de Docker personalizada para App Service**: permite usar contenedores de Docker desde un registro de contenedor o un contenedor de cliente al implementar una aplicación web en Linux.

    >**Cuándo se debe usar**: al implementar una aplicación web en Linux en una imagen de Docker.

    >**Para comenzar**: [use una imagen de Docker personalizada para App Service en Linux](../../app-service/containers/quickstart-docker-go.md)

### <a name="authentication"></a>Autenticación

Es fundamental saber no solo quién está usando las aplicaciones, sino también evitar el acceso no autorizado a los recursos. Azure proporciona varias maneras de autenticar los clientes de las aplicaciones.

-   **Azure Active Directory (Azure AD)**: el servicio de administración de acceso e identidades de Microsoft, basado en la nube y multiinquilino. Puede agregar inicio de sesión único (SSO) a sus aplicaciones mediante la integración con Azure AD. Para acceder a las propiedades del directorio, puede usar Graph API de Azure AD directamente o Microsoft Graph API. Puede integrar en Azure AD compatibilidad para el entorno de autorización OAuth2.0 y Open ID Connect mediante puntos de conexión HTTP/REST nativos y las bibliotecas de autenticación de Azure AD multiplataforma.

    >**Cuándo se debe usar**: cuando quiera proporcionar una experiencia SSO, trabajar con datos basados en Graph o autenticar usuarios basados en dominio.

    >**Para comenzar**: para más información, consulte la [guía para desarrolladores de Azure Active Directory](../../active-directory/develop/v1-overview.md).

-   **Autenticación de App Service**: al elegir App Service para hospedar la aplicación, también obtiene compatibilidad con la autenticación integrada para Azure AD, además de proveedores de identidades sociales, como Facebook, Google, Microsoft y Twitter.

    >**Cuándo se debe usar**: cuando quiera habilitar la autenticación en una aplicación de App Service con Azure AD, proveedores de identidades sociales, o ambos.

    >**Para comenzar**: para más información sobre la autenticación en App Service, consulte [Autenticación y autorización en Azure App Service](../../app-service/overview-authentication-authorization.md).

Para más información sobre los procedimientos recomendados de seguridad en Azure, consulte [Patrones y procedimientos recomendados de seguridad en Azure](../../security/security-best-practices-and-patterns.md).

### <a name="monitoring"></a>Supervisión

Cuando la aplicación está lista y funcionando en Azure, debe poder supervisar el rendimiento, controlar los problemas y ver cómo los clientes usan la aplicación. Azure ofrece varias opciones de supervisión.

-   **Visual Studio Application Insights**: servicio de análisis extensible hospedado en Azure que se integra con Visual Studio para supervisar las aplicaciones web activas. Proporciona los datos que necesita para mejorar continuamente el rendimiento y el uso de las aplicaciones, tanto si está hospedadas en Azure como si no.

    >**Para comenzar**: siga el [tutorial de Application Insights](../../azure-monitor/app/app-insights-overview.md).

-   **Azure Monitor**: servicio que ayuda a visualizar, consultar, enrutar, archivar y actuar sobre las métricas y los registros generados por la infraestructura y los recursos de Azure. Monitor proporciona las vistas de datos que se ven en Azure Portal y es un origen único para la supervisión de los recursos de Azure.
 
    >**Para comenzar**: [inicie su andadura por Azure Monitor](../../monitoring-and-diagnostics/monitoring-get-started.md).

### <a name="devops-integration"></a>Integración con DevOps

Tanto para aprovisionar máquinas virtuales como para publicar aplicaciones web con integración continua, Azure se integra con la mayoría de las herramientas de DevOps más conocidas. Es compatible con herramientas como Jenkins, GitHub, Puppet, Chef, TeamCity, Ansible, Azure DevOps y otras, y permite trabajar con las herramientas que ya tiene y maximizar su experiencia actual.

>**Pruébelo ya:** [pruebe algunas de las integraciones con DevOps](https://azure.microsoft.com/try/devops/).

>**Para comenzar**: para ver las opciones de DevOps para una aplicación de App Service, consulte [Implementación continua en Azure App Service](../../app-service/deploy-continuous-deployment.md).


## <a name="azure-regions"></a>Regiones de Azure

Azure es una plataforma en la nube global que está disponible con carácter general en muchas regiones de todo el mundo. Al aprovisionar un servicio, una aplicación o una máquina virtual en Azure, se le pedirá que seleccione una región, que representa un centro de datos específico en el que se ejecuta la aplicación o donde se almacenan los datos. Estas regiones se corresponden con ubicaciones específicas, que se publican en la página [Regiones de Azure](https://azure.microsoft.com/regions/).

### <a name="choose-the-best-region-for-your-application-and-data"></a>Elección de la región indicada para la aplicación y los datos

Una de las ventajas de usar Azure es que puede implementar aplicaciones en distintos centros de datos de todo el mundo. La región que elija puede afectar al rendimiento de la aplicación. Por ejemplo, es mejor elegir la región que esté más cerca de la mayoría de sus clientes para reducir la latencia de las solicitudes de red. Quizás también quiera seleccionar una región para cumplir los requisitos legales para distribuir la aplicación en determinados países. Siempre es recomendable almacenar los datos de la aplicación en el mismo centro de datos o en un centro de datos lo más cercano posible al centro de datos donde se hospeda la aplicación.

### <a name="multi-region-apps"></a>Aplicaciones para varias regiones

Aunque no es muy probable, es posible que un centro de datos completo pase a estar sin conexión debido a sucesos tales como desastres naturales o errores de Internet. Es recomendable hospedar las aplicaciones empresariales vitales en más de un centro de datos para proporcionar la máxima disponibilidad. El uso de varias regiones también reduce la latencia para los usuarios globales y ofrece más flexibilidad a la hora de actualizar las aplicaciones.

Algunos servicios, como Virtual Machines y App Services, usan [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) para habilitar la compatibilidad con varias regiones con conmutación por error entre regiones con el fin de admitir aplicaciones empresariales de alta disponibilidad. Para obtener un ejemplo, consulte [Arquitectura de referencia de Azure: Ejecución de una aplicación web en varias regiones](https://docs.microsoft.com/azure/architecture/reference-architectures/app-service-web-app/multi-region).

>**Cuándo se debe usar**: si tiene aplicaciones empresariales y de alta disponibilidad que se benefician de la conmutación por error y la replicación.

## <a name="how-do-i-manage-my-applications-and-projects"></a>¿Cómo administro mis aplicaciones y proyectos?

Azure ofrece un amplio conjunto de experiencias para crear y administrar recursos de Azure, aplicaciones y proyectos, tanto mediante programación como en [Azure Portal](https://portal.azure.com/).

### <a name="command-line-interfaces-and-powershell"></a>Interfaces de línea de comandos y PowerShell

Azure proporciona dos mecanismos para administrar las aplicaciones y los servicios desde la línea de comandos con Bash, Terminal, el símbolo del sistema o la herramienta de línea de comandos que prefiera. Por lo general, puede realizar las mismas tareas desde la línea de comandos y en Azure Portal, por ejemplo, crear y configurar máquinas virtuales, redes virtuales, aplicaciones web y otros servicios.

-   [Interfaz de la línea de comandos (CLI) de Azure](../../xplat-cli-install.md): permite conectarse a una suscripción de Azure y programar diversas tareas para los recursos de Azure desde la línea de comandos.

-   [Azure PowerShell](../../powershell-install-configure.md): proporciona un conjunto de módulos con cmdlets que permiten administrar los recursos de Azure mediante Windows PowerShell.

### <a name="azure-portal"></a>Azure Portal

Azure Portal es una aplicación basada en web que puede usarse para crear, administrar y eliminar recursos y servicios de Azure. Azure Portal se encuentra en <https://portal.azure.com>. Incluye un panel personalizable, herramientas para administrar los recursos de Azure y acceso a la configuración de la suscripción y a la información de facturación. Para más información, consulte la [Introducción a Azure Portal](../../azure-portal-overview.md).

### <a name="rest-apis"></a>API de REST

Azure se basa en un conjunto de API de REST que dan soporte a la interfaz de usuario de Azure Portal. La mayoría de estas API de REST también se admiten para aprovisionar y administrar mediante programación los recursos y las aplicaciones de Azure desde cualquier dispositivo con acceso a Internet. Para obtener la documentación del conjunto completo de API de REST, consulte la [referencia del SDK de REST de Azure](https://docs.microsoft.com/rest/api/).

### <a name="apis"></a>API existentes

Además de las API de REST, muchos servicios de Azure también permiten administrar mediante programación los recursos desde las aplicaciones mediante SDK de Azure específicos de la plataforma, incluidos los SDK para las siguientes plataformas de desarrollo:

-   [.NET](https://go.microsoft.com/fwlink/?linkid=834925)
-   [Node.js](https://docs.microsoft.com/javascript/azure)
-   [Java](https://docs.microsoft.com/java/azure)
-   [PHP](https://github.com/Azure/azure-sdk-for-php/blob/master/README.md)
-   [Python](https://docs.microsoft.com/python/azure)
-   [Ruby](https://github.com/Azure/azure-sdk-for-ruby/blob/master/README.md)
-   [Go](https://docs.microsoft.com/go/azure)

Servicios como [Mobile Apps](../../app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library.md) y [Azure Media Services](../../media-services/previous/media-services-dotnet-how-to-use.md) proporcionan SDK de cliente para que pueda acceder a los servicios desde aplicaciones de cliente móviles y web.

### <a name="azure-resource-manager"></a>Azure Resource Manager 
    
Ejecutar una aplicación en Azure suele implicar trabajar con varios servicios de Azure, todos los cuales siguen el mismo ciclo de vida y pueden considerarse como una unidad lógica. Por ejemplo, una aplicación web podría usar los servicios Web Apps, SQL Database, Storage, Azure Redis Cache y Azure Content Delivery Network. [Azure Resource Manager](../../azure-resource-manager/resource-group-overview.md) permite trabajar con los recursos de la aplicación como un grupo. Todos los recursos se pueden implementar, actualizar o eliminar en una sola operación coordinada.

Además de agrupar y administrar recursos relacionados de forma lógica, Azure Resource Manager incluye funcionalidades de implementación que permiten personalizar la implementación y configuración de los recursos relacionados. Por ejemplo, con Resource Manager puede implementar y configurar una aplicación que consta de varias máquinas virtuales, un equilibrador de carga y una instancia de Azure SQL Database como una sola unidad.

Estas implementaciones se desarrollan usando una plantilla de Azure Resource Manager, que es un documento con formato JSON. Las plantillas permiten definir una implementación y administrar las aplicaciones mediante plantillas declarativas en lugar de scripts. Las plantillas pueden funcionar en diferentes entornos, como pruebas, almacenamiento provisional y producción. Por ejemplo, con las plantillas puede agregar un botón a un repositorio de GitHub que implementa el código en el repositorio para un conjunto de servicios de Azure con un solo clic.

>**Cuándo se debe usar**: las plantillas de Resource Manager se usan cuando se desea una implementación basada en plantillas de la aplicación que se pueda administrar mediante programación con las API REST, la CLI de Azure y Azure PowerShell.

>**Para comenzar**: para empezar a trabajar con plantillas, consulte cómo [crear plantillas de Azure Resource Manager](../../resource-group-authoring-templates.md).

## <a name="understanding-accounts-subscriptions-and-billing"></a>Descripción de las cuentas, suscripciones y facturación

Como a los desarrolladores, nos gusta bucear en el código e intentar que nuestras aplicaciones se pongan en marcha lo más rápidamente posible. Queremos que pueda a empezar a trabajar con Azure tan fácilmente como sea posible. Para ello, Azure ofrece una [evaluación gratuita](https://azure.microsoft.com/free/). Algunos servicios incluso tienen una funcionalidad "Pruébelo gratis", como [Azure App Service](https://tryappservice.azure.com/), que ni siquiera requiere crear una cuenta. Lanzarse a codificar e implementar su aplicación en Azure es tan divertido como importante es tomarse algún tiempo para comprender el funcionamiento de Azure desde la perspectiva de las cuentas de usuario, las suscripciones y la facturación.

### <a name="what-is-an-azure-account"></a>¿Qué es una cuenta de Azure?

Para poder crear o trabajar con una suscripción de Azure, debe tener una cuenta de Azure. Una cuenta de Azure es simplemente una identidad en Azure AD o en un directorio, por ejemplo, una organización profesional o académica, que sea de confianza para Azure AD. Si no pertenece a una organización de este tipo, siempre puede crear una suscripción usando su cuenta Microsoft, que es de confianza para Azure AD. Para más información sobre la integración de directorios locales de Windows Server Active Directory con Azure AD, consulte [Integración de los directorios locales con Azure Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md).

Cada suscripción de Azure tiene una relación de confianza con una instancia de Azure AD. Esto significa que confía en ese directorio para autenticar usuarios, servicios y dispositivos. Varias suscripciones pueden confiar en el mismo directorio, pero una suscripción confía solo en un único directorio. Para más información, consulte [Asociación de las suscripciones de Azure con Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

Además de definir identidades de cuenta de Azure individuales, también llamadas *usuarios*, también puede definir *grupos* en Azure AD. Crear grupos de usuarios es una buena manera de administrar el acceso a los recursos de una suscripción mediante el control de acceso basado en roles (RBAC). Para ver cómo crear grupos, consulte [Creación de un grupo y adición de miembros en Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). También puede crear y administrar grupos [con PowerShell](../../active-directory/users-groups-roles/groups-settings-v2-cmdlets.md).

### <a name="manage-your-subscriptions"></a>Administración de suscripciones

Una suscripción es una agrupación lógica de servicios de Azure que está vinculada a una cuenta de Azure. Una única cuenta de Azure puede contener varias suscripciones. La facturación de los servicios de Azure se realiza por suscripción. Para obtener una lista de las ofertas de suscripción disponibles por tipo, consulte [Detalles de las ofertas de Microsoft Azure](https://azure.microsoft.com/support/legal/offer-details/). Las suscripciones de Azure tienen un administrador de cuenta, que tiene control total sobre la suscripción y un administrador de servicios, que tiene control sobre todos los servicios de la suscripción. Para saber más sobre los administradores de suscripción clásica, vea [Agregar o cambiar los administradores de la suscripción de Azure](../../billing/billing-add-change-azure-subscription-administrator.md). Además de a los administradores, se puede conceder control detallado de los recursos de Azure con [control de acceso basado en rol (RBAC)](../../role-based-access-control/overview.md) a cuentas individuales.

#### <a name="resource-groups"></a>Grupos de recursos

Al aprovisionar nuevos servicios de Azure, puede hacerlo en una suscripción determinada. Los servicios de Azure individuales, a los que también se llama recursos, se crean en el contexto de un grupo de recursos. Los grupos de recursos facilitan la implementación y administración de los recursos de una aplicación. Un grupo de recursos debe contener todos los recursos de la aplicación con los que desea trabajar como una unidad. Puede mover los recursos entre diferentes grupo de recursos e, incluso, entre diferentes suscripciones. Para más información sobre cómo mover recursos, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../resource-group-move-resources.md).

El Explorador de recursos de Azure es una fantástica herramienta para visualizar los recursos que ya haya creado en su suscripción. Para más información, consulte [Uso del Explorador de recursos de Azure para ver y modificar recursos](../../resource-manager-resource-explorer.md).

#### <a name="grant-access-to-resources"></a>Concesión de acceso a los recursos

Al permitir el acceso a los recursos de Azure, siempre es recomendable proporcionar a los usuarios los privilegios mínimos que se necesiten para realizar una tarea determinada.

-   **Control de acceso basado en rol (RBAC)**: en Azure, puede conceder acceso a cuentas de usuario (entidades de seguridad) en un ámbito especificado: suscripción, grupo de recursos o recurso individual. RBAC permite implementar un conjunto de recursos en un grupo de recursos y conceder permisos a un usuario específico o a un grupo. También permite limitar el acceso únicamente a los recursos que pertenecen al grupo de recursos de destino. También puede conceder acceso a un único recurso, por ejemplo, una máquina virtual o una red virtual. Para conceder acceso, asigne un rol al usuario, grupo de usuarios o entidad de servicio. Hay muchos roles predefinidos y puede definir también sus propios roles personalizados. Para obtener más información, vea [¿Qué es el control de acceso basado en rol (RBAC)?](../../role-based-access-control/overview.md)

    >**Cuándo se debe usar**: cuando se necesite administración de acceso detallada para usuarios y grupos o cuando sea necesario convertir a un usuario en propietario de una suscripción.

    >**Para comenzar**: para más información, consulte [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).

-   **Objetos de entidad de servicio**: además de proporcionar acceso a las entidades de seguridad de usuarios y grupos, puede conceder el mismo acceso a una entidad de servicio.

    > **Cuándo se debe usar**: al administrar los recursos de Azure o conceder acceso a las aplicaciones mediante programación. Para más información, consulte [Creación de aplicación de Active Directory y una entidad de servicio](../../active-directory/develop/howto-create-service-principal-portal.md).

#### <a name="tags"></a>Etiquetas

Azure Resource Manager permite asignar etiquetas personalizadas a los recursos individuales. Las etiquetas, que son pares de clave-valor, pueden resultar útiles si necesita organizar los recursos para facturación o supervisión. Las etiquetas son una manera de realizar un seguimiento de los recursos a través de varios grupos de recursos. Puede asignar las etiquetas en el portal, en la plantilla de Azure Resource Manager o mediante programación, con la API de REST, la CLI de Azure o PowerShell. Puede asignar varias etiquetas a cada recurso. Para más información, consulte [Uso de etiquetas para organizar los recursos de Azure](../../resource-group-using-tags.md).

### <a name="billing"></a>Facturación

En el traslado de la informática local a los servicios hospedados en la nube, el cálculo y seguimiento del uso del servicio, así como los costos relacionados, son cuestiones importantes. Es importante poder calcular el costo mensual de la ejecución de los nuevos recursos. También debe poder predecir la facturación de un mes determinado en función del gasto actual.

#### <a name="get-resource-usage-data"></a>Obtención de datos de uso de recursos

Azure proporciona un conjunto de API de REST de facturación que dan acceso a información sobre los metadatos y el consumo de recursos de las suscripciones de Azure. Estas API de facturación permiten predecir y administrar mejor los costos de Azure. Puede realizar un seguimiento y analizar el gasto en incrementos de una hora, crear alertas de gastos y predecir la facturación futura en función de las tendencias de uso actuales.

>**Para comenzar**: para más información sobre el uso de las API de facturación, consulte la [introducción a las API Billing Usage y RateCard de Azure](../../billing-usage-rate-card-overview.md).

#### <a name="predict-future-costs"></a>Predicción de los costos futuros

Aunque resulta difícil calcular los costos de antemano, Azure tiene una [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) que permite calcular el costo de los recursos implementados. También puede usar la hoja de facturación en el portal y las API de REST de facturación para calcular los costos futuros en función del consumo actual.

>**Para comenzar**: consulte [introducción a las API Billing Usage y RateCard de Azure](../../billing-usage-rate-card-overview.md).
