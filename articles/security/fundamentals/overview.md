---
title: Introducción a la seguridad de Azure | Microsoft Docs
description: Aprenda acerca de la seguridad de Azure, sus servicios y cómo funciona.
services: security
documentationcenter: na
author: UnifyCloud
manager: rkarlin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/18/2019
ms.author: TomSh
ms.openlocfilehash: 9c107789fe4a88b3cd73a2f290871016b95cae19
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371403"
---
# <a name="introduction-to-azure-security"></a>Introducción a la seguridad de Azure
## <a name="overview"></a>Información general
Sabemos que la seguridad tiene la máxima prioridad en la nube y conocemos la importancia que tiene que buscar información exacta y a tiempo sobre la seguridad de Azure. Una de las mejores razones para usar Azure en sus aplicaciones y servicios es poder aprovechar su amplia gama de funcionalidades y herramientas de seguridad. Estas herramientas y funcionalidades permiten crear soluciones seguras en la plataforma Azure segura. Microsoft Azure proporciona confidencialidad, integridad y disponibilidad para los datos del cliente, al mismo tiempo que hace posible una responsabilidad transparente.

En este artículo se proporciona una visión completa de la seguridad disponible con Azure.

### <a name="azure-platform"></a>Plataforma Azure
Azure es una plataforma de servicios en la nube pública que admite una amplia selección de sistemas operativos, lenguajes de programación, plataformas, herramientas, bases de datos y dispositivos. Puede ejecutar contenedores de Linux con integración de Docker, compilar aplicaciones con JavaScript, Python, .NET, PHP, Java, Node.js y crear back-ends para dispositivos iOS, Android y Windows.

Los servicios en la nube pública de Azure admiten las mismas tecnologías en las que ya confían millones de desarrolladores y profesionales de TI. Al crear en un proveedor de servicios en la nube pública o al migrar recursos de TI a uno, confía en las capacidades de la organización para proteger sus aplicaciones y datos con los servicios y los controles que facilitan para administrar la seguridad de sus recursos en la nube.

La infraestructura de Azure está diseñada desde la instalación hasta las aplicaciones para hospedar millones de clientes simultáneamente, y proporciona una base de confianza en la que las empresas pueden satisfacer sus requisitos de seguridad.

Además, Azure ofrece una amplia gama de opciones de seguridad configurables, así como la capacidad de controlarlas, por lo que puede personalizar la seguridad para satisfacer los requisitos exclusivos de las implementaciones de su organización. Este documento explica cómo las funcionalidades de seguridad de Azure pueden ayudarle a cumplir estos requisitos.

> [!Note]
> El objetivo principal de este documento se centra en los controles orientados al cliente que puede usar para personalizar y aumentar la seguridad de sus aplicaciones y servicios.
>
> Para información sobre cómo Microsoft protege la plataforma de Azure, consulte [Seguridad de la infraestructura de Azure](infrastructure.md).

## <a name="summary-of-azure-security-capabilities"></a>Resumen de las funcionalidades de seguridad de Azure

### <a name="features-to-secure-the-azure-platform"></a>Características para proteger la plataforma Azure
Las características siguientes son funcionalidades que puede examinar para tener la garantía de que la plataforma Azure se administra de forma segura. Se han proporcionado vínculos para que vea con más detalle cómo Microsoft aborda las cuestiones de confianza del cliente en cuatro áreas: plataforma segura, privacidad y controles, cumplimiento normativo y transparencia.

| [Plataforma segura](https://www.microsoft.com/trustcenter/Security/default.aspx)  | [Privacidad y controles](https://www.microsoft.com/trustcenter/Privacy/default.aspx)  |[Cumplimiento normativo](https://www.microsoft.com/trustcenter/Compliance/default.aspx)   | [Transparencia](https://www.microsoft.com/trustcenter/Transparency/default.aspx) |
| :-- | :-- | :-- | :-- |
| [Ciclo de desarrollo de seguridad](https://www.microsoft.com/sdl/), auditorías internas | [Administración constante de los datos](https://www.microsoft.com/trustcenter/Privacy/You-own-your-data) | [Centro de confianza](https://www.microsoft.com/trustcenter/default.aspx) |[Cómo Microsoft protege los datos de clientes en servicios de Azure](https://www.microsoft.com/trustcenter/Transparency/default.aspx) |
| [Entrenamiento de seguridad obligatorio, comprobación de antecedentes](https://downloads.cloudsecurityalliance.org/star/self-assessment/StandardResponsetoRequestforInformationWindowsAzureSecurityPrivacy.docx) |  [Control en la ubicación de los datos](https://www.microsoft.com/trustcenter/Privacy/Where-your-data-is-located) |  [Centro de controles comunes](https://www.microsoft.com/trustcenter/Common-Controls-Hub) |[Cómo Microsoft administra la ubicación de datos en los servicios de Azure](https://azuredatacentermap.azurewebsites.net/)|
| [Pruebas de penetración](https://downloads.cloudsecurityalliance.org/star/self-assessment/StandardResponsetoRequestforInformationWindowsAzureSecurityPrivacy.docx), [detección de intrusiones, DDoS](https://www.microsoft.com/trustcenter/Security/ThreatManagement), [auditorías y registro](https://www.microsoft.com/trustcenter/Security/AuditingAndLogging) | [Proporcionar acceso a datos en sus propios términos](https://www.microsoft.com/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms) |  [Lista de comprobación de diligencia debida para Cloud Services](https://www.microsoft.com/trustcenter/Compliance/Due-Diligence-Checklist) |[Personas de Microsoft que pueden acceder a sus datos y bajo qué términos](https://www.microsoft.com/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms)|
| [Centro de datos de vanguardia](https://www.microsoft.com/cloud-platform/global-datacenters), seguridad física, [red segura](network-overview.md) | [Respuesta a las autoridades judiciales](https://www.microsoft.com/trustcenter/Privacy/Responding-to-govt-agency-requests-for-customer-data) |  [Cumplimiento por servicio, ubicación y sector](https://www.microsoft.com/trustcenter/Compliance/default.aspx) |[Cómo Microsoft protege los datos de clientes en servicios de Azure](https://www.microsoft.com/trustcenter/Transparency/default.aspx)|
|  [Respuesta a incidentes de seguridad](https://aka.ms/SecurityResponsepaper), [responsabilidad compartida](https://aka.ms/sharedresponsibility) |[Rigurosos estándares de privacidad](https://www.microsoft.com/TrustCenter/Privacy/We-set-and-adhere-to-stringent-standards) |  | [Revisión de la certificación para servicios de Azure, centro de transparencia](https://www.microsoft.com/trustcenter/Compliance/default.aspx)|

### <a name="features-to-secure-data-and-application"></a>Características para proteger los datos y las aplicaciones
Según el modelo de servicio en la nube, hay diferencias de en quién recae la responsabilidad de administrar la seguridad de la aplicación o el servicio. Existen funcionalidades en la plataforma Azure para ayudarle a satisfacer estas responsabilidades mediante características integradas y soluciones de asociados que se pueden implementar en una suscripción de Azure.

Las funcionalidades integradas se organizan en seis áreas funcionales: Operaciones, Aplicaciones, Almacenamiento, Redes, Proceso e Identidades. Se proporcionan detalles adicionales sobre las características y funcionalidades disponibles en la plataforma Azure para estas seis áreas de forma resumida.

## <a name="operations"></a>Operaciones
En esta sección se proporciona información adicional acerca de características fundamentales para las operaciones de seguridad y un resumen de estas funcionalidades.

### <a name="security-and-audit-dashboard"></a>Panel Security and Audit
La solución [Security and Audit](../../security-center/security-center-intro.md) ofrece una vista completa de la situación de seguridad de TI de su organización con [consultas de búsqueda integradas](https://blogs.technet.microsoft.com/msoms/2016/01/21/easy-microsoft-operations-management-suite-search-queries/) para problemas importantes que requieren su atención. El panel de [Security and Audit](https://technet.microsoft.com/library/mt484091.aspx) es la pantalla principal para todo lo relacionado con la seguridad en los registros de Azure Monitor. Proporciona información de alto nivel sobre el estado de seguridad de los equipos. También incluye la capacidad de ver todos los eventos de las últimas 24 horas, 7 días o cualquier otro intervalo personalizado.

Además, puede configurar Security & Compliance para [llevar a cabo automáticamente acciones específicas](https://blogs.technet.microsoft.com/robdavies/2016/04/20/simple-look-at-oms-alert-remediation-with-runbooks-part-1/) cuando se detecte un evento concreto.

### <a name="azure-resource-manager"></a>Azure Resource Manager
[Azure Resource Manager](../../azure-resource-manager/management/deployment-models.md) permite trabajar con los recursos de la solución como grupo. Todos los recursos de la solución se pueden implementar, actualizar o eliminar en una sola operación coordinada. Use una [plantilla de Azure Resource Manager](https://blogs.technet.microsoft.com/canitpro/2015/06/29/devops-basics-infrastructure-as-code-arm-templates/) para la implementación; esta puede funcionar en distintos entornos, como producción, pruebas y ensayo. Administrador de recursos proporciona funciones de seguridad, auditoría y etiquetado que le ayudan a administrar los recursos después de la implementación.

Las implementaciones basadas en plantillas de Azure Resource Manager ayudan a mejorar la seguridad de las soluciones implementadas en Azure porque incluyen una configuración de control de seguridad estándar y pueden integrarse en implementaciones basadas en plantillas estandarizadas. Esto reduce el riesgo de que se produzcan errores de configuración de seguridad, posibles durante las implementaciones manuales.

### <a name="application-insights"></a>Application Insights
[Application Insights](https://docs.microsoft.com/azure/application-insights/) es un servicio de Application Performance Management (APM) extensible para desarrolladores web. Con Application Insights, puede supervisar sus aplicaciones web en directo y detectar automáticamente anomalías de rendimiento. Incluye herramientas de análisis eficaces que le ayudan a diagnosticar problemas y comprender lo que hacen realmente los usuarios con la aplicación. Supervisa la aplicación durante todo el tiempo que se ejecute, tanto durante las pruebas como después de haberla publicado o implementado.

Application Insights crea gráficos y tablas que muestran, por ejemplo, en qué horas del día se obtiene la mayoría de los usuarios, la capacidad de respuesta de la aplicación y lo bien que la atienden los servicios externos de los que depende.

Si hay bloqueos, errores o problemas de rendimiento, puede buscar en los datos de la telemetría para diagnosticar la causa. Además, el servicio le envía mensajes de correo electrónico si se produce cualquier cambio en la disponibilidad y el rendimiento de la aplicación. Por tanto, Application Insights se convierte en una valiosa herramienta de seguridad porque ayuda con la disponibilidad, parte de la tríada de seguridad formada también por la confidencialidad y la integridad.

### <a name="azure-monitor"></a>Azure Monitor
[Azure Monitor](https://docs.microsoft.com/azure/monitoring-and-diagnostics/) ofrece funciones de visualización, consulta, enrutamiento, alertas, escalado automático y automatización de los datos tanto de la infraestructura de Azure ([registro de actividad](../../azure-monitor/platform/platform-logs-overview.md)) como de cada recurso individual de Azure ([registros de diagnóstico](../../azure-monitor/platform/platform-logs-overview.md)). Puede usar Azure Monitor para que le alerte sobre eventos relacionados con la seguridad que se generen en registros de Azure.

### <a name="azure-monitor-logs"></a>Registros de Azure Monitor
[Registros de Azure Monitor](https://azure.microsoft.com/documentation/services/log-analytics/): ofrece una solución de administración de TI tanto para infraestructura local como para la basada en la nube de terceros (como AWS), además de recursos de Azure. Los datos de Azure Monitor se pueden enrutar directamente a los registros de Azure Monitor para poder ver los registros y las métricas de todo el entorno en un único lugar.

Los registros de Azure Monitor pueden ser una herramienta útil en el análisis forense y otros análisis de seguridad, ya que permiten buscar rápidamente entre grandes cantidades de entradas relacionadas con la seguridad siguiendo un enfoque de consulta flexible. Además, los [registros de proxy y firewall locales se pueden exportar a Azure y poner a disposición para su análisis con registros de Azure Monitor.](../../log-analytics/log-analytics-agent-windows.md)

### <a name="azure-advisor"></a>Azure Advisor
[Azure Advisor](../../advisor/index.yml) es un consultor personalizado en la nube que le ayudará a optimizar las implementaciones de Azure. Analiza la telemetría de uso y configuración de los recursos y, posteriormente, recomienda soluciones que ayudan a mejorar el [rendimiento](../../advisor/advisor-performance-recommendations.md), la [seguridad](../../advisor/advisor-security-recommendations.md) y la [alta disponibilidad](../../advisor/advisor-high-availability-recommendations.md) de los recursos, al mismo tiempo que busca oportunidades para [reducir el gasto general en Azure](../../advisor/advisor-cost-recommendations.md). Azure Advisor proporciona recomendaciones de seguridad, que pueden mejorar de forma notable la posición general de seguridad para las soluciones que se implementan en Azure. Estas recomendaciones se extraen del análisis de seguridad realizado por [Azure Security Center.](../../security-center/security-center-intro.md)

### <a name="azure-security-center"></a>Azure Security Center
[Security Center](../../security-center/security-center-intro.md) ayuda a evitar amenazas y a detectarlas y responder a ellas con más visibilidad y control sobre la seguridad de los recursos de Azure. Proporciona administración de directivas y supervisión de la seguridad integrada en las suscripciones de Azure, ayuda a detectar las amenazas que podrían pasar desapercibidas y funciona con un amplio ecosistema de soluciones de seguridad.

Además, Security Center le ayuda con las operaciones de seguridad, al proporcionarle un único panel donde aparecen alertas y recomendaciones sobre las que puede actuar de inmediato. A menudo, puede corregir problemas con solo hacer clic dentro de la consola de Security Center.
## <a name="applications"></a>APLICACIONES
En esta sección se proporciona información adicional acerca de características fundamentales para la seguridad de las aplicaciones y un resumen de estas funcionalidades.

### <a name="web-application-vulnerability-scanning"></a>Examen de vulnerabilidades para aplicaciones web
Una de las maneras más fáciles de empezar a probar si existen puntos vulnerables en su [aplicación de App Service](../../app-service/overview.md) consiste en usar la [integración con Tinfoil Security](https://azure.microsoft.com/blog/web-vulnerability-scanning-for-azure-app-service-powered-by-tinfoil-security/) para realizar un examen con un solo clic del nivel de vulnerabilidad de su aplicación. Puede ver los resultados de la prueba en un informe fácil de entender, y aprender a corregir cada vulnerabilidad con instrucciones detalladas.

### <a name="penetration-testing"></a>Pruebas de penetración
Si prefiere realizar sus propias pruebas de penetración o desea usar otro conjunto de aplicaciones de análisis u otro proveedor, debe seguir el [proceso de aprobación de pruebas de penetración de Azure](https://docs.microsoft.com/azure/security/fundamentals/pen-testing ) y obtener aprobación previa para realizar las pruebas de penetración deseadas.

### <a name="web-application-firewall"></a>Firewall de aplicaciones web
El firewall de aplicaciones web (WAF) de [Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) protege las aplicaciones web ante ataques web habituales, como inyección de código SQL, ataques de scripts entre sitios y secuestros de sesiones. Viene preconfigurado con protección frente a las amenazas identificadas por el [Proyecto abierto de seguridad de aplicaciones web (OWASP) como las 10 vulnerabilidades más comunes](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project).

### <a name="authentication-and-authorization-in-azure-app-service"></a>Autenticación y autorización en Azure App Service
La [autenticación/autorización de App Service](../../app-service/overview-authentication-authorization.md) es una característica que proporciona una forma para que la aplicación lleve a cabo el inicio de sesión de usuarios sin necesidad de cambiar el código en el back-end de aplicación. Ofrece una forma fácil de proteger su aplicación y trabajar con datos por usuario.

### <a name="layered-security-architecture"></a>Arquitectura de seguridad por niveles
Como los [entornos de App Service](../../app-service/environment/app-service-app-service-environment-intro.md) proporcionan un entorno en tiempo de ejecución aislado que está implementado en una [instancia de Azure Virtual Network](../../virtual-network/virtual-networks-overview.md), los desarrolladores pueden crear una arquitectura de seguridad por niveles que proporcione diferentes niveles de acceso a la red para cada capa de aplicación. Un objetivo común es ocultar los back-ends de API al acceso general desde Internet y permitir que solo las aplicaciones web ascendentes puedan llamar a las API. Los [grupos de seguridad de red (NSG)](https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/) pueden usarse en subredes de Azure Virtual Network que contengan entornos de App Service para restringir el acceso público a aplicaciones API.

### <a name="web-server-diagnostics-and-application-diagnostics"></a>Diagnóstico del servidor web y diagnóstico de aplicaciones
Aplicaciones web de App Service ofrece la funcionalidad de diagnóstico para registrar información del servidor web y de la aplicación web. De forma lógica, estos diagnósticos se dividen en [diagnósticos del servidor web](../../app-service/troubleshoot-diagnostic-logs.md) y [diagnóstico de aplicaciones](https://technet.microsoft.com/library/hh530058(v=sc.12).aspx). El servidor web incluye dos avances importantes para el diagnóstico y la solución de problemas de sitios y aplicaciones.

La primera característica nueva es la información de estado en tiempo real sobre grupos de aplicaciones, procesos de trabajo, sitios, dominios de aplicación y solicitudes en ejecución. La segunda ventaja nueva son los eventos de seguimiento detallados que siguen una solicitud a lo largo del proceso completo de solicitud y respuesta.

Para habilitar la recopilación de estos eventos de seguimiento, se puede configurar IIS 7 para que capture automáticamente registros de seguimiento completos, en formato XML, para cualquier solicitud determinada en función del tiempo transcurrido o de códigos de respuesta de error.

#### <a name="web-server-diagnostics"></a>Diagnósticos del servidor web
Puede habilitar o deshabilitar los siguientes tipos de registros:

-   Registro de errores detallado: registra información detallada de errores para códigos de estado HTTP que indican un problema (código de estado 400 o superior). Puede contener información que puede ayudar a determinar por qué el servidor ha devuelto el código de error.

-   Seguimiento de solicitudes con error: registra información detallada acerca de solicitudes con error, incluido un seguimiento de los componentes de IIS usados para procesar la solicitud y el tiempo dedicado a cada componente. Esto puede resultar útil si trata de aumentar el rendimiento del sitio o de aislar lo que causa la devolución de un error HTTP específico.

-   Registro del servidor web: registra información sobre todas las transacciones HTTP con el formato de archivo de registro extendido de W3C. Este informe resulta útil para determinar las métricas totales del sitio, como el número de solicitudes tramitadas o cuántas solicitudes proceden de una dirección IP específica.

#### <a name="application-diagnostics"></a>Diagnósticos de aplicaciones
El [diagnóstico de aplicaciones](../../app-service/troubleshoot-diagnostic-logs.md) le permite capturar información generada por una aplicación web. Las aplicaciones de ASP.NET pueden usar la clase [System.Diagnostics.Trace](https://msdn.microsoft.com/library/system.diagnostics.trace) para registrar información en el registro de diagnóstico de la aplicación. En Diagnóstico de aplicaciones, hay dos tipos principales de eventos, los relacionados con el rendimiento de las aplicaciones y los relacionados con los errores de las aplicaciones. Los errores se pueden subdividir en problemas de conectividad, seguridad y funcionamiento. Los problemas de funcionamiento normalmente guardan relación con un problema con el código de la aplicación.

En Diagnóstico de aplicaciones, puede ver los eventos agrupados de las siguientes maneras:

-   Todos (muestra todos los eventos)
-   Errores de aplicación (muestra eventos de excepción)
-   Rendimiento (muestra eventos de rendimiento)

## <a name="storage"></a>Storage
En esta sección se proporciona información adicional acerca de características fundamentales para la seguridad del almacenamiento de Azure y un resumen de estas funcionalidades.

### <a name="role-based-access-control-rbac"></a>Control de acceso basado en rol (RBAC)
Puede proteger su cuenta de almacenamiento con el control de acceso basado en rol (RBAC). En organizaciones que quieren aplicar directivas de seguridad para el acceso a los datos, es imprescindible restringir el acceso en función de los principios de seguridad [necesidad de saber](https://en.wikipedia.org/wiki/Need_to_know) y [mínimo privilegio](https://en.wikipedia.org/wiki/Principle_of_least_privilege). Estos derechos de acceso se conceden al asignar el rol RBAC adecuado a grupos y aplicaciones de un determinado ámbito. Puede aprovechar los [roles integrados de Azure](../../role-based-access-control/built-in-roles.md), tales como el de colaborador de la cuenta de almacenamiento, para asignar privilegios a los usuarios. El acceso a las claves de almacenamiento para una cuenta de almacenamiento mediante el modelo de [Azure Resource Manager](../../storage/blobs/security-recommendations.md) puede controlarse con el control de acceso basado en rol (RBAC).

### <a name="shared-access-signature"></a>Firma de acceso compartido
Una [firma de acceso compartido (SAS)](../../storage/common/storage-dotnet-shared-access-signature-part-1.md) ofrece acceso delegado a los recursos en la cuenta de almacenamiento. Esto significa que puede conceder a un cliente permisos limitados para objetos en su cuenta de almacenamiento durante un período específico y con un conjunto determinado de permisos sin tener que compartir las claves de acceso a las cuentas.

### <a name="encryption-in-transit"></a>Cifrado en tránsito
Cifrado en tránsito es un mecanismo para proteger datos cuando se transmiten a través de redes. Con Azure Storage, puede proteger los datos mediante:
-   [Cifrado de nivel de transporte](../../storage/blobs/security-recommendations.md), como HTTPS para transferir datos a Azure Storage o desde este servicio.

-   [Cifrado en el cable](../../storage/blobs/security-recommendations.md), como el [cifrado SMB 3.0](../../storage/blobs/security-recommendations.md) para [recursos compartidos de Azure File](../../storage/files/storage-dotnet-how-to-use-files.md).

-   Cifrado en el cliente, para cifrar los datos antes de transferirlos al almacenamiento y descifrarlos una vez transferidos desde el almacenamiento.

### <a name="encryption-at-rest"></a>Cifrado en reposo
Para muchas organizaciones, el cifrado de los datos en reposo es un paso obligatorio en lo que respecta a la privacidad de los datos, el cumplimiento y la soberanía de los datos. Hay tres características de seguridad del almacenamiento de Azure que proporcionan cifrado de datos "en reposo":

-   [Cifrado del servicio de almacenamiento](../../storage/common/storage-service-encryption.md) permite solicitar que el servicio de almacenamiento cifre automáticamente los datos al escribirlos en Azure Storage.

-   [Cifrado de cliente](../../storage/common/storage-client-side-encryption.md) también proporciona la característica de cifrado en reposo.

-   [Azure Disk Encryption](../azure-security-disk-encryption-overview.md) permite cifrar los discos de datos y del sistema operativo usados por una máquina virtual de IaaS.

### <a name="storage-analytics"></a>Storage Analytics
[Azure Storage Analytics](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics) realiza el registro y proporciona datos de métricas para una cuenta de almacenamiento. Puede usar estos datos para hacer un seguimiento de solicitudes, analizar tendencias de uso y diagnosticar problemas con la cuenta de almacenamiento. Storage Analytics registra información detallada sobre las solicitudes correctas y erróneas realizadas a un servicio de almacenamiento. Esta información se puede utilizar para supervisar solicitudes concretas y para diagnosticar problemas con un servicio de almacenamiento. Las solicitudes se registran en función de la mejor opción. Se registran los siguientes tipos de solicitudes autenticadas:
-   Solicitudes correctas

-   Solicitudes erróneas, incluyendo errores de tiempo de espera, de limitación, de red, de autorización, etc

-   Solicitudes que utilizan una firma de acceso compartido (SAS), incluyendo las solicitudes correctas y las erróneas

-   Solicitudes de datos de análisis

### <a name="enabling-browser-based-clients-using-cors"></a>Habilitación de clientes basados en explorador mediante CORS
El [uso compartido de recursos entre orígenes (CORS)](https://docs.microsoft.com/rest/api/storageservices/fileservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services) es un mecanismo que permite que los dominios se concedan permisos entre sí para acceder a los recursos de los demás. El agente de usuario envía encabezados adicionales para asegurarse de que el código JavaScript que se carga desde un determinado dominio tenga permiso para acceder a recursos ubicados en otro dominio. Después, el último dominio responde con encabezados adicionales para permitir o denegar al dominio original el acceso a sus recursos.

Los servicios de almacenamiento de Azure ahora admiten CORS, así que, una vez establecidas las reglas de CORS para el servicio, una solicitud autenticada correctamente realizada en el servicio desde un dominio diferente se evalúa para determinar si se permite según las reglas que ha especificado.

## <a name="networking"></a>Redes
En esta sección se proporciona información adicional acerca de características fundamentales para la seguridad de red de Azure y un resumen de estas funcionalidades.

### <a name="network-layer-controls"></a>Controles de nivel de red
El control de acceso de red es el acto de limitar la conectividad entre subredes o dispositivos específicos y constituye el núcleo de la seguridad de red. El objetivo es garantizar que las máquinas virtuales y los servicios sean accesibles solo para los usuarios y dispositivos pertinentes.

#### <a name="network-security-groups"></a>Grupos de seguridad de red
Un [grupo de seguridad de red (NSG)](../../virtual-network/virtual-network-vnet-plan-design-arm.md) es un firewall de filtrado de paquetes básico con estado que le permite controlar el acceso basado en una [5-tupla](https://www.techopedia.com/definition/28190/5-tuple). Los NSG no proporcionan inspección de nivel de aplicación ni controles de acceso autenticados. Se pueden usar para controlar el tráfico que se mueve entre subredes dentro de una instancia de Azure Virtual Network y el tráfico entre una instancia de Azure Virtual Network e Internet.

#### <a name="route-control-and-forced-tunneling"></a>Control de ruta y tunelización forzada
La posibilidad de controlar el comportamiento de enrutamiento en Azure Virtual Network es una funcionalidad crítica del control de acceso y la seguridad de red. Por ejemplo, si desea asegurarse de que todo el tráfico que entre en su instancia de Azure Virtual Network y salga de ella pase por ese dispositivo de seguridad virtual, debe ser capaz de controlar y personalizar el comportamiento de enrutamiento. Para ello, puede configurar rutas definidas por el usuario en Azure.

Las [rutas definidas por el usuario](../../virtual-network/virtual-networks-udr-overview.md) le permiten personalizar rutas de acceso entrantes y salientes para el tráfico que entra y sale de máquinas virtuales individuales o subredes para garantizar la ruta más segura posible. [tunelización forzada](https://www.petri.com/azure-forced-tunneling) es un mecanismo que puede usar para tener la seguridad de que no se permite que sus servicios inicien una conexión con dispositivos en Internet.

Esto es diferente a poder aceptar conexiones entrantes y luego responder a ellas. En este caso, los servidores front-end web tienen que responder a solicitudes de los hosts de Internet, así que se permite la entrada en estos servidores web del tráfico cuyo origen es Internet y dichos servidores pueden responder.

La tunelización forzada normalmente se usa para forzar que el tráfico saliente hacia Internet atraviese firewalls y servidores proxy de seguridad locales.

#### <a name="virtual-network-security-appliances"></a>Dispositivos de seguridad de red virtual
Aunque los grupos de seguridad de red, las rutas definidas por el usuario y la tunelización forzada proporcionan un nivel de seguridad en las capas de red y transporte del [modelo OSI](https://en.wikipedia.org/wiki/OSI_model), habrá ocasiones en las que desee habilitar la seguridad en niveles más altos de la pila. Puede acceder a estas características mejoradas de seguridad de red mediante una solución de dispositivo de seguridad de red de asociados de Azure. Para encontrar las soluciones de seguridad de red de asociados más actuales de Azure, visite [Azure Marketplace](https://azure.microsoft.com/marketplace/) y busque "seguridad" y "seguridad de la red".

### <a name="azure-virtual-network"></a>Azure Virtual Network
Una red virtual de Azure (VNet) es una representación de su propia red en la nube. Es un aislamiento lógico del tejido de red de Azure dedicado a su suscripción. Puede controlar por completo los bloques de direcciones IP, la configuración DNS, las directivas de seguridad y las tablas de rutas dentro de esta red. Puede segmentar la red virtual en subredes y colocar máquinas virtuales de IaaS de Azure o [servicios en la nube (instancias de rol de PaaS)](../../cloud-services/cloud-services-choose-me.md) en instancias de Azure Virtual Network.

Además, puede conectar la red virtual a su red local mediante una de las [opciones de conectividad](../../vpn-gateway/index.yml) disponibles en Azure. En esencia, puede ampliar su red en Azure, con control total sobre bloques de direcciones IP con la ventaja de la escala empresarial que ofrece Azure.

Las redes de Azure admiten diversos escenarios de acceso remoto seguro. Algunos son:

-   [Conexión de estaciones de trabajo individuales a una instancia de Azure Virtual Network](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

-   [Conexión de la red local a una instancia de Azure Virtual Network con una VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md)

-   [Conexión de la red local a una instancia de Azure Virtual Network con un vínculo WAN dedicado](../../expressroute/expressroute-introduction.md)

-   [Conexión de instancias de Azure Virtual Network entre sí](../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

### <a name="vpn-gateway"></a>VPN Gateway
Para enviar tráfico de red entre su instancia de Azure Virtual Network y el sitio local, es preciso que cree una puerta de enlace de VPN para la instancia de Azure Virtual Network. Una [puerta de enlace de VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md) es un tipo de puerta de enlace de red virtual que envía tráfico cifrado a través de una conexión pública. También puede utilizar puertas de enlace de VPN para enviar tráfico entre instancias de Azure Virtual Network a través del tejido de red de Azure.

### <a name="express-route"></a>ExpressRoute
Microsoft Azure [ExpressRoute](../../expressroute/expressroute-introduction.md) es un vínculo WAN dedicado que le permite extender sus redes locales a Microsoft Cloud a través de una conexión privada y dedicada que facilita un proveedor de conectividad.

![ExpressRoute](./media/overview/azure-security-fig1.png)

Con ExpressRoute, se pueden establecer conexiones con servicios en la nube de Microsoft, como Microsoft Azure, Microsoft 365 y CRM Online. La conectividad puede ser desde una red de conectividad universal (IP VPN), una red Ethernet de punto a punto, o una conexión cruzada virtual a través de un proveedor de conectividad en una instalación de ubicación compartida.

Las conexiones ExpressRoute no pasan por Internet y, por tanto, se pueden considerar más seguras que las soluciones basadas en VPN. Esto permite a las conexiones de ExpressRoute ofrecer más confiabilidad, más velocidad, menor latencia y mayor seguridad que las conexiones normales a través de Internet.


### <a name="application-gateway"></a>Application Gateway
Microsoft [Azure Application Gateway](../../application-gateway/overview.md) cuenta con un [controlador de entrega de aplicaciones (ADC)](https://en.wikipedia.org/wiki/Application_delivery_controller) que se ofrece como servicio y que proporciona varias funcionalidades de equilibrio de carga de nivel 7 para la aplicación.

![Application Gateway](./media/overview/azure-security-fig2.png)

Le permite optimizar la productividad de las granjas de servidores web al traspasar la carga de la terminación TLS con mayor actividad de la CPU a Application Gateway (lo que también se conoce como "descarga TLS" o "puente TLS"). Además, dispone de otras funcionalidades de enrutamiento de nivel 7, como la distribución round robin del tráfico entrante, la afinidad de sesiones basada en cookies, el enrutamiento basado en rutas de acceso URL y la capacidad de hospedar varios sitios web tras una única instancia de Application Gateway. Azure Application Gateway es un equilibrador de carga de nivel 7.

Proporciona conmutación por error, solicitudes HTTP de enrutamiento de rendimiento entre distintos servidores, independientemente de que se encuentren en la nube o en una implementación local.

La aplicación proporciona numerosas características de controlador de entrega de aplicaciones (ADC), entre las que se incluyen el equilibrio de carga HTTP, la afinidad de sesiones basada en cookies, la [descarga TLS](../../application-gateway/tutorial-restrict-web-traffic-powershell.md), los sondeos personalizados sobre el estado y la compatibilidad con sitios múltiples.

### <a name="web-application-firewall"></a>Firewall de aplicaciones web
El firewall de aplicaciones web (WAF) es una característica de [Azure Application Gateway](../../application-gateway/overview.md) diseñada para proteger las aplicaciones web que utilizan la puerta de enlace de aplicaciones para funciones estándar de Application Delivery Control (ADC). El firewall de aplicaciones web ofrece protección contra las diez vulnerabilidades web más comunes identificadas por OWASP.

![Firewall de aplicaciones web](./media/overview/azure-security-fig1.png)

-   Protección contra la inyección de código SQL

-   Protección contra ataques web comunes, como inyección de comandos, contrabando de solicitudes HTTP, división de respuestas HTTP y ataque remoto de inclusión de archivos

-   Protección contra infracciones del protocolo HTTP

-   Protección contra anomalías del protocolo HTTP, como la falta de agentes de usuario de host y encabezados de aceptación

-   Prevención contra bots, rastreadores y escáneres

-   Detección de errores de configuración comunes en aplicaciones (es decir, Apache, IIS, etc.)

Disponer de un firewall de aplicaciones web centralizado que ofrezca protección contra los ataques web facilita enormemente la administración de la seguridad y proporciona mayor protección a la aplicación contra amenazas de intrusiones. Las soluciones de WAF también pueden reaccionar más rápido ante una amenaza de la seguridad aplicando revisiones que aborden una vulnerabilidad conocida en una ubicación central en lugar de proteger cada una de las aplicaciones web por separado. Las puertas de enlace de aplicaciones existentes pueden transformarse rápidamente en puertas de enlace con un firewall de aplicaciones web.

### <a name="traffic-manager"></a>Traffic Manager
Microsoft [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) permite controlar la distribución del tráfico de los usuarios para puntos de conexión de servicio en distintos centros de datos. Entre los puntos de conexión de servicio compatibles con Traffic Manager, se incluyen máquinas virtuales de Azure, Web Apps y servicios en la nube. También puede utilizar el Administrador de tráfico con puntos de conexión externos, que no forman parte de Azure. Traffic Manager usa el sistema de nombres de dominio (DNS) para dirigir las solicitudes del cliente al punto de conexión más adecuado en función de un [método de enrutamiento del tráfico](../../traffic-manager/traffic-manager-routing-methods.md) y el estado de los puntos de conexión.

Traffic Manager proporciona una serie de métodos de enrutamiento del tráfico para satisfacer las necesidades de distintas aplicaciones, la [supervisión](../../traffic-manager/traffic-manager-monitoring.md) del estado de los puntos de conexión y la conmutación automática por error. Traffic Manager es resistente a errores, incluidos los que afecten a toda una región de Azure.

### <a name="azure-load-balancer"></a>Azure Load Balancer
[Azure Load Balancer](../../load-balancer/load-balancer-overview.md) proporciona una alta disponibilidad y un elevado rendimiento de red a las aplicaciones. Se trata de un equilibrador de carga de nivel 4 (TCP y UDP) que distribuye el tráfico entrante entre las instancias de servicio correctas de los servicios que se definen en un conjunto de carga equilibrada. Azure Load Balancer puede configurarse para lo siguiente:

-   Equilibrar la carga del tráfico entrante de Internet entre las máquinas virtuales. Esta configuración se conoce como [equilibrio de carga público](../../load-balancer/components.md#frontend-ip-configurations).

-   Equilibrar la carga del tráfico entre máquinas virtuales de una red virtual, entre máquinas virtuales de servicios en la nube o entre equipos locales y máquinas virtuales de una red virtual entre entornos locales. Esta configuración se conoce como " [equilibrio de carga interno](../../load-balancer/components.md#frontend-ip-configurations)".

- Reenvío de tráfico externo a una máquina virtual determinada

### <a name="internal-dns"></a>DNS interno
Puede administrar la lista de servidores DNS usados en una red virtual en el Portal de administración o en el archivo de configuración de red. Un cliente puede agregar hasta 12 servidores DNS para cada red virtual. Al especificar servidores DNS, es importante comprobar que se enumeren los servidores DNS del cliente en el orden correcto para el entorno del cliente. Las listas de servidores DNS no funcionan con Round Robin. Se utilizan en el orden en que se especifican. Si se puede acceder al primer servidor DNS de la lista, el cliente usa ese servidor DNS con independencia de si el servidor DNS funciona correctamente o no. Para cambiar el orden del servidor DNS para la red virtual del cliente, quite los servidores DNS de la lista y agréguelos en el orden en que el cliente desee. DNS es compatible con el aspecto de disponibilidad, parte de la tríada de seguridad formada también por la confidencialidad y la integridad.

### <a name="azure-dns"></a>Azure DNS
El [sistema de nombres de dominio](https://technet.microsoft.com/library/bb629410.aspx), o DNS, es responsable de traducir (o resolver) el nombre del sitio web o del servicio en su dirección IP. [Azure DNS](../../dns/dns-overview.md) es un servicio de hospedaje para dominios DNS que permite resolver nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure. DNS es compatible con el aspecto de disponibilidad, parte de la tríada de seguridad formada también por la confidencialidad y la integridad.

### <a name="azure-monitor-logs-nsgs"></a>Grupos de seguridad de red de registros de Azure Monitor
Puede habilitar las siguientes categorías de registro de diagnóstico para los NSG:

-   Evento: contiene entradas para las que se aplican reglas de NSG a las máquinas virtuales y a los roles de instancia en función de la dirección MAC. El estado de estas reglas se recopila cada 60 segundos.

-   Contador de regla: contiene entradas para el número de veces que se aplica cada regla NSG para denegar o permitir el tráfico.

### <a name="security-center"></a>Security Center
[Azure Security Center](../../security-center/security-center-intro.md) analiza continuamente el estado de seguridad de los recursos de Azure para los procedimientos recomendados de seguridad de red. Cuando Security Center identifica posibles vulnerabilidades de seguridad, crea [recomendaciones](../../security-center/security-center-recommendations.md) que lo guiarán por el proceso de configuración de los controles necesarios para reforzar y proteger sus recursos.

## <a name="compute"></a>Proceso
En esta sección se proporciona información adicional acerca de características fundamentales para esta área y un resumen de estas funcionalidades.

### <a name="antimalware--antivirus"></a>Antimalware y antivirus
Con IaaS de Azure, puede usar software antimalware de proveedores de seguridad como Microsoft, Symantec, Trend Micro, McAfee y Kaspersky, para proteger las máquinas virtuales de archivos malintencionados, adware y otras amenazas. [Microsoft Antimalware](antimalware.md) para Azure Cloud Services y Virtual Machines es una funcionalidad de protección que permite identificar y eliminar virus, spyware y otro software malintencionado. Microsoft Antimalware activa alertas configurables cuando software no deseado o malintencionado intenta instalarse o ejecutarse en los sistemas de Azure. Microsoft Antimalware también puede implementarse mediante Azure Security Center.

### <a name="hardware-security-module"></a>Módulo de seguridad de hardware
La autenticación y el cifrado no mejoran la seguridad a menos que las propias claves estén protegidas. Puede simplificar la administración y la seguridad de claves y secretos críticos guardándolos en [Azure Key Vault](../../key-vault/general/overview.md). Key Vault permite guardar claves en módulos de seguridad de hardware (HSM) que tienen la certificación FIPS 140-2 nivel 2. Sus claves de cifrado de SQL Server para copias de seguridad o [cifrado de datos transparente](https://msdn.microsoft.com/library/bb934049.aspx) se pueden almacenar en Key Vault con otras claves y secretos de sus aplicaciones. Los permisos y el acceso a estos elementos protegidos se administran con [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/).

### <a name="virtual-machine-backup"></a>Copia de seguridad de máquina virtual
[Azure Backup](../../backup/backup-overview.md) es una solución que protege los datos de su aplicación sin necesidad de realizar ninguna inversión y afrontando unos costos operativos mínimos. Los errores de una aplicación pueden dañar los datos, y los errores humanos pueden crear errores en las aplicaciones que conlleven problemas de seguridad. Con Azure Backup, las máquinas virtuales que ejecutan Windows y Linux están protegidas.

### <a name="azure-site-recovery"></a>Azure Site Recovery
Una parte importante de la estrategia de [continuidad empresarial/recuperación ante desastres (BCDR)](../../best-practices-availability-paired-regions.md) de su organización es averiguar cómo mantener las aplicaciones y cargas de trabajo corporativas en funcionamiento cuando se produzcan interrupciones planeadas e imprevistas. [Azure Site Recovery](../../site-recovery/site-recovery-overview.md) ayuda a coordinar la replicación, la conmutación por error y la recuperación de aplicaciones y cargas de trabajo para que estén disponibles desde una ubicación secundaria si la ubicación principal deja de funcionar.

### <a name="sql-vm-tde"></a>TDE de máquina virtual de SQL
El [cifrado de datos transparente (TDE)](../../virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-keyvault.md) y cifrado de nivel de columna (CLE) son características de cifrado de SQL Server. Esta forma de cifrado requiere que los clientes administren y almacenen las claves criptográficas que se usan para el cifrado.

El servicio Azure Key Vault (AKV) está diseñado para mejorar la seguridad y la administración de estas claves en una ubicación segura y con gran disponibilidad. El Conector de SQL Server permite que SQL Server use estas claves desde Azure Key Vault.

Si ejecuta SQL Server con máquinas locales, hay una serie de pasos que puede seguir para acceder a Azure Key Vault desde la instancia de SQL Server local. Pero para SQL Server en las máquinas virtuales de Azure, puede ahorrar tiempo si usa la característica Integración de Azure Key Vault. Con algunos cmdlets de Azure PowerShell para habilitar esta característica, puede automatizar la configuración necesaria para que una máquina virtual de SQL tenga acceso a su Almacén de claves.

### <a name="vm-disk-encryption"></a>Cifrado de discos de máquinas virtuales
[Azure Disk Encryption](../azure-security-disk-encryption-overview.md) es una nueva funcionalidad que permite cifrar los discos de las máquinas virtuales de IaaS Windows y Linux. Usa la característica BitLocker estándar del sector de Windows y la característica DM-Crypt de Linux para ofrecer cifrado de volumen para el sistema operativo y los discos de datos. La solución se integra con Azure Key Vault para ayudarle a controlar y administrar los secretos y las claves de cifrado de discos en su suscripción de Key Vault. La solución también garantiza que todos los datos de los discos de máquinas virtuales se cifran en reposo en Azure Storage.

### <a name="virtual-networking"></a>Redes virtuales
Las máquinas virtuales necesita conectividad de red. Para satisfacer este requisito, es necesario que Azure Virtual Machines esté conectado a una instancia de Azure Virtual Network. Azure Virtual Network es una construcción lógica creada encima del tejido de red físico de Azure. Cada instancia de [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md) lógica está aislada de todas las demás instancias de Azure Virtual Network. Este aislamiento contribuye a garantizar que otros clientes de Microsoft Azure no puedan acceder al tráfico de red de sus implementaciones.

### <a name="patch-updates"></a>Actualizaciones de revisiones
Las actualizaciones de revisiones proporcionan la base para encontrar y corregir posibles problemas y simplificar el proceso de administración de actualizaciones de software al reducir el número de actualizaciones de software que debe implementar en su empresa y aumentar la capacidad de supervisar el cumplimiento.

### <a name="security-policy-management-and-reporting"></a>Informes y administración de directivas de seguridad
[Security Center](../../security-center/security-center-intro.md) ayuda a evitar amenazas y a detectarlas y responder a ellas, y proporciona una mayor visibilidad y control sobre la seguridad de sus recursos de Azure. Proporciona administración de directivas y supervisión de la seguridad integradas en las suscripciones de Azure, ayuda a detectar amenazas que podrían pasar desapercibidas y funciona con un amplio ecosistema de soluciones de seguridad.

## <a name="identity-and-access-management"></a>Administración de identidades y acceso
La protección de los sistemas, las aplicaciones y los datos comienza por los controles de acceso basado en identidad. Las características de administración de identidades y acceso que están integradas en los servicios y productos de Microsoft para empresas ayudan a proteger su información personal y profesional ante el acceso no autorizado al mismo tiempo que se pone a disposición de los usuarios legítimos cuando y donde la necesiten.

### <a name="secure-identity"></a>Protección de la identidad
Microsoft utiliza varias tecnologías y procedimientos recomendados de seguridad en sus productos y servicios para administrar las identidades y el acceso.

-   [Multi-Factor Authentication](https://azure.microsoft.com/services/multi-factor-authentication/) requiere que los usuarios utilicen varios métodos para el acceso, tanto localmente como en la nube. Proporciona autenticación sólida con una variedad de sencillas opciones de verificación, a la vez que admite usuarios mediante un proceso de inicio de sesión simple.

-   [Microsoft Authenticator](https://aka.ms/authenticator) ofrece una experiencia de Multi-Factor Authentication fácil de usar que funciona tanto con Microsoft Azure Active Directory como con cuentas de Microsoft e incluye compatibilidad con ponibles y aprobaciones basadas en huellas digitales.

-   La [aplicación de directivas de contraseña](https://azure.microsoft.com/documentation/articles/active-directory-passwords-policy/) aumenta la seguridad de las contraseñas tradicionales al imponer requisitos de longitud y complejidad, la rotación periódica obligatoria y el bloqueo de cuenta después de intentos de autenticación incorrectos.

-   La [autenticación basada en tokens](https://azure.microsoft.com/documentation/articles/active-directory-authentication-scenarios/) habilita la autenticación a través de Azure Active Directory.

-   El [control de acceso basado en rol (RBAC)](https://azure.microsoft.com/documentation/articles/role-based-access-built-in-roles/) permite conceder acceso en función del rol asignado al usuario, lo que hace más fácil proporcionar a los usuarios únicamente la cantidad de acceso que necesitan para realizar sus tareas. Puede personalizar RBAC según el modelo de negocio de su organización y su tolerancia al riesgo.

-   La [administración de identidades integrada (identidad híbrida)](https://azure.microsoft.com/documentation/articles/active-directory-hybrid-identity-design-considerations-overview/) le permite mantener el control del acceso de los usuarios en centros de datos internos y plataformas en la nube, al crear una única identidad de usuario para la autenticación y autorización en todos los recursos.

### <a name="secure-apps-and-data"></a>Protección de aplicaciones y datos
[Azure Active Directory](https://azure.microsoft.com/services/active-directory/), una solución en la nube de administración integral de identidades y acceso, ayuda a proteger el acceso a los datos en aplicaciones locales y en la nube, además de simplificar la administración de usuarios y grupos. Combina servicios de directorio centrales, gobernanza avanzada de identidades, seguridad y administración del acceso a aplicaciones; además, facilita a los desarrolladores la integración en sus aplicaciones de la administración de identidades basada en directivas. Para mejorar su instancia de Azure Active Directory, puede agregar funcionalidades de pago con las ediciones Azure Active Directory Basic, Premium P1 y Premium P2.

| Características comunes/gratuitas     | Características de la edición Basic    |Características de la edición Premium P1 |Características de la edición Premium P2 | Azure Active Directory Join, solo características relacionadas con Windows 10|
| :------------- | :------------- |:------------- |:------------- |:------------- |
|   [Objetos de directorio](../../active-directory/active-directory-whatis.md), [administración de usuarios y grupos (agregar, actualizar y eliminar), aprovisionamiento basado en el usuario, registro de dispositivos](../../active-directory/active-directory-whatis.md), [inicio de sesión único (SSO)](../../active-directory/active-directory-whatis.md), [autoservicio de cambio de contraseña para usuarios de la nube](../../active-directory/active-directory-whatis.md), [conexión (motor de sincronización que extiende los directorios locales a Azure Active Directory)](../../active-directory/active-directory-whatis.md), [informes de seguridad y uso](../../active-directory/active-directory-whatis.md)       |   [Aprovisionamiento y administración del acceso basados en grupo](../../active-directory/active-directory-whatis.md), [autoservicio de restablecimiento de contraseña para usuarios de la nube](../../active-directory/active-directory-whatis.md), [personalización de marca de la compañía (personalización de las páginas de inicio de sesión y del panel de acceso)](../../active-directory/active-directory-whatis.md), [proxy de aplicación](../../active-directory/active-directory-whatis.md), [Acuerdo de Nivel de Servicio del 99,9 %](../../active-directory/active-directory-whatis.md) |  [Autoservicio de administración de grupos y aplicaciones, autoservicio de incorporación de aplicaciones o grupos dinámicos](../../active-directory/active-directory-whatis.md), [autoservicio de restablecimiento de contraseña, cambio o desbloqueo con escritura diferida local](../../active-directory/active-directory-whatis.md), [Multi-Factor Authentication (en la nube y local [Servidor MFA])](../../active-directory/active-directory-whatis.md), [CAL de MIM + servidor MIM](../../active-directory/active-directory-whatis.md), [Cloud App Discovery](../../active-directory/active-directory-whatis.md), [Connect Health](../../active-directory/active-directory-whatis.md), [Sustitución automática de la contraseña para cuentas de grupo](../../active-directory/active-directory-whatis.md)|    [Identity Protection](../../active-directory/identity-protection/overview.md), [Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md)|   [Unir un dispositivo a Azure AD, SSO de escritorio, Microsoft Passport para Azure AD, recuperación de BitLocker de administrador](../../active-directory/active-directory-whatis.md), [inscripción automática de MDM, autoservicio de recuperación de BitLocker, administradores locales adicionales para dispositivos Windows 10 a través de la unión a Azure AD](../../active-directory/active-directory-whatis.md)|


- [Cloud App Discovery](../../active-directory/cloudappdiscovery-get-started.md) es una característica Premium de Azure Active Directory que permite identificar aplicaciones en la nube usadas por los empleados de su organización.

- [Azure Active Directory Identity Protection](https://azure.microsoft.com/documentation/articles/active-directory-identityprotection/) es un servicio de seguridad que usa las funcionalidades de detección de anomalías de Azure Active Directory para proporcionar una vista consolidada de detecciones de riesgo y vulnerabilidades potenciales que podrían afectar a las identidades de su organización.

- [Azure Active Directory Domain Services](https://azure.microsoft.com/services/active-directory-ds/) permite unir máquinas virtuales de Azure a un dominio sin necesidad de implementar controladores de dominio. Los usuarios inician sesión en estas máquinas virtuales usando sus credenciales corporativas de Active Directory y pueden acceder a los recursos sin problemas.

- [Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) es un servicio de administración de identidades global y de alta disponibilidad para aplicaciones orientadas al consumidor que pueden utilizar hasta cientos de millones de identidades e integrarse en plataformas web y móviles. Los clientes pueden iniciar sesión en todas las aplicaciones mediante experiencias personalizables que usan cuentas de redes sociales existentes o pueden crear credenciales independientes.

- [Colaboración B2B de Azure Active Directory](https://aka.ms/aad-b2b-collaboration) es una solución segura de integración de asociados que admite relaciones entre empresas al permitir que los asociados accedan a las aplicaciones corporativas y a los datos de forma selectiva mediante sus identidades autoadministradas.

- [Azure Active Directory Join](https://azure.microsoft.com/documentation/articles/active-directory-azureadjoin-overview/) permite extender las funcionalidades de nube a dispositivos Windows 10 para la administración centralizada. Permite que los usuarios se conecten a la nube corporativa o de la organización a través de Azure Active Directory y simplifica el acceso a aplicaciones y recursos.

- La característica [Proxy de la aplicación de Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-application-proxy-get-started/) proporciona inicio de sesión único (SSO) y acceso remoto seguro para aplicaciones web hospedadas de forma local.

## <a name="next-steps"></a>Pasos siguientes

- Comprenda la [responsabilidad compartida en la nube](shared-responsibility.md).

- Aprenda cómo [Azure Security Center](https://azure.microsoft.com/services/security-center/) le ayuda a evitar amenazas y a detectarlas y responder a ellas, gracias a la mayor visibilidad y control sobre la seguridad de todos sus recursos de Azure.
