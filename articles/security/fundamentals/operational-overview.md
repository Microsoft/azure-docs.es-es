---
title: Información general sobre la seguridad operativa de Azure| Microsoft Docs
description: Obtenga información sobre la seguridad operativa de Azure en esta introducción. La seguridad operativa hace referencia a los servicios, los controles y las características de protección de recursos.
services: security
documentationcenter: na
author: unifycloud
manager: rkarlin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/31/2019
ms.author: tomsh
ms.openlocfilehash: f3ec284de3d43e75bd0832b1d98bb9d319a7ca0e
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87925318"
---
# <a name="azure-operational-security-overview"></a>Información general sobre la seguridad operativa de Azure

Con [seguridad operativa de Azure](/azure/security/fundamentals/operational-security), se hace referencia a los servicios, los controles y las características disponibles para los usuarios para proteger sus datos, aplicaciones y otros recursos en Microsoft Azure. Es un marco que incorpora el conocimiento adquirido a través de una variedad de funcionalidades exclusivas de Microsoft. Estas funcionalidades incluyen el Ciclo de vida de desarrollo de seguridad (SDL) de Microsoft, el programa Microsoft Security Response Center y un conocimiento profundo del panorama de amenazas de ciberseguridad.

## <a name="azure-management-services"></a>Servicios de administración de Azure

Un equipo de operaciones de TI es el responsable de administrar la infraestructura del centro de datos, las aplicaciones y los datos, incluida la estabilidad y la seguridad de estos sistemas. Sin embargo, la obtención de información de seguridad sobre el cada vez mayor número de entornos de TI complejos, a menudo requiere que las organizaciones reúnan datos a partir de distintos sistemas de administración y seguridad.

Los [registros de Microsoft Azure Monitor](/azure/operations-management-suite/operations-management-suite-overview) es una solución de administración de TI basada en la nube de Microsoft que le permite administrar y proteger su infraestructura local y en la nube. Los siguientes servicios que se ejecutan en Azure proporcionan su funcionalidad principal. En Azure, se incluyen varios servicios que le permiten administrar y proteger la infraestructura local y en la nube. Cada servicio proporciona una función de administración específica. Puede combinar los servicios para lograr distintos escenarios de administración. 

### <a name="azure-monitor"></a>Azure Monitor

[Azure Monitor](/azure/azure-monitor/overview) recopila datos de orígenes administrados en almacenes de datos centralizados. Estos datos pueden incluir eventos, datos de rendimiento o datos personalizados proporcionados mediante la API. Una vez recopilados los datos, están disponibles para las alertas, el análisis y la exportación.

Puede consolidar datos de varios orígenes y combinar datos de los servicios de Azure con el entorno local existente. Los registros de Azure Monitor también separan claramente la recopilación de los datos de la acción realizada en los datos para que todas las acciones estén disponibles para todos los tipos de datos.

### <a name="automation"></a>Automation

[Azure Automation](/azure/automation/automation-intro) le ofrece una manera de automatizar las tareas manuales, propensas a errores, con una ejecución prolongada y repetidas con frecuencia que se realizan normalmente en un entorno empresarial y en la nube. Ahorra tiempo y aumenta la confiabilidad de las tareas administrativas. Incluso programa estas tareas para que se realicen automáticamente a intervalos regulares. Puede automatizar procesos mediante runbooks o automatizar la administración de configuración mediante la configuración de estado deseada.

### <a name="backup"></a>Copia de seguridad

[Azure Backup](/azure/backup/backup-introduction-to-azure-backup) es el servicio de Azure que puede usar para hacer una copia de seguridad de los datos (protegerlos) y restaurarlos en Microsoft Cloud. Azure Backup reemplaza su solución de copia de seguridad local o remota existente por una solución confiable, segura y rentable basada en la nube.

Azure Backup ofrece componentes que se descargan e implementan en el equipo o servidor adecuados, o en la nube. El componente, o agente, que se implemente depende de lo que quiera proteger. Todos los componentes de Azure Backup (tanto si va a proteger los datos de forma local como en la nube) se pueden usar para realizar una copia de seguridad de datos en un almacén de Azure Recovery Services en Azure.

Para obtener más información, consulte la [tabla de componentes de Azure Backup](/azure/backup/backup-overview#what-can-i-back-up).

### <a name="site-recovery"></a>Site Recovery

[Azure Site Recovery](https://azure.microsoft.com/documentation/services/site-recovery) proporciona continuidad del negocio a través de la organización de la replicación de máquinas virtuales y físicas locales en Azure o en un sitio secundario. Si su sitio primario no está disponible, se realizará la conmutación por error a la ubicación secundaria para que los usuarios pueden seguir trabajando. Se realizará una conmutación por recuperación cuando los sistemas vuelvan a las condiciones de funcionamiento normales. Use Azure Security Center para realizar una detección de amenazas más inteligente y eficaz.

## <a name="azure-active-directory"></a>Azure Active Directory

[Azure Active Directory (Azure AD)](/azure/active-directory/active-directory-enable-sso-scenario) es un servicio de identidad completo que:

-   Habilita la Administración de identidad y acceso (IAM) como servicio en la nube.
-   Proporciona administración de acceso central, inicio de sesión único (SSO) y creación de informes.
-   Admite la administración de acceso integrado para [miles de aplicaciones](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory) de Azure Marketplace, como Salesforce, Google Apps, Box y Concur.

Azure AD también incluye un conjunto completo de [funcionalidades de administración de identidades](/azure/security/security-identity-management-overview#security-monitoring-alerts-and-machine-learning-based-reports), como las siguientes:

- [Autenticación multifactor](/azure/multi-factor-authentication/multi-factor-authentication)
- [Administración de contraseñas de autoservicio](https://azure.microsoft.com/resources/videos/self-service-password-reset-azure-ad/)
- [Administración de grupos de autoservicio](/azure/active-directory/active-directory-passwords-update-your-own-password)
- [Administración de cuentas con privilegios](/azure/active-directory/active-directory-privileged-identity-management-configure)
- [Control de acceso basado en roles de Azure (Azure RBAC)](/azure/role-based-access-control/overview)
- [Supervisión del uso de la aplicación](/azure/active-directory/connect-health/active-directory-aadconnect-health)
- [Auditoría avanzada](/azure/active-directory/active-directory-reporting-activity-audit-logs)
- [Supervisión de seguridad y alertas](/azure/operations-management-suite/oms-security-responding-alerts)

Con Azure Active Directory, todas las aplicaciones que publica para sus asociados y clientes (empresa o consumidor) tendrán las mismas funcionalidades de administración de identidad y acceso. Esto permite reducir significativamente los costos operativos.

## <a name="azure-security-center"></a>Azure Security Center

[Azure Security Center](/azure/security-center/security-center-intro) ayuda a evitar, detectar y responder a amenazas con más visibilidad y control sobre la seguridad de sus recursos de Azure. Proporciona una supervisión de la seguridad y una administración de directivas integradas en sus suscripciones. Le ayuda a detectar amenazas que podrían pasar desapercibidas, y funciona con un amplio ecosistema de soluciones de seguridad.

[Proteja los datos de máquinas virtuales (VM)](/azure/security-center/security-center-linux-virtual-machine) en Azure proporcionando visibilidad a la configuración de seguridad de su máquina virtual y supervisando las amenazas. Security Center puede supervisar las máquinas virtuales con los siguientes fines:

- Configuración de seguridad del sistema operativo con las reglas de configuración recomendadas.
- Seguridad del sistema y actualizaciones críticas que faltan.
- Recomendaciones de protección de puntos de conexión.
- Validación de cifrado de disco.
- Ataques basados en la red.

Security Center usa el [control de acceso basado en roles (RBAC de Azure)](/azure/role-based-access-control/role-assignments-portal). RBAC proporciona [roles integrados](../../role-based-access-control/built-in-roles.md) que se pueden asignar a usuarios, grupos y servicios de Azure.

Security Center evalúa la configuración de los recursos para identificar problemas de seguridad y vulnerabilidades. En Security Center, solo se muestra información relacionada con un recurso cuando tiene asignado el rol de propietario, colaborador o lector para la suscripción o grupo de recursos al que pertenece un recurso.

>[!Note]
>Consulte [Permisos en Azure Security Center](/azure/security-center/security-center-permissions) para obtener más información sobre los roles y las acciones permitidas en Security Center.

Security Center usa Microsoft Monitoring Agent. Es el mismo agente que usa el servicio de Azure Monitor. Los datos que recopila este agente se almacenan en una [área de trabajo](/azure/log-analytics/log-analytics-manage-access) existente de Log Analytics asociada con la suscripción a Azure o en una nueva área de trabajo, según la geolocalización de la VM.

## <a name="azure-monitor"></a>Azure Monitor

Los problemas de rendimiento de la aplicación en la nube pueden afectar a su negocio. Con varios componentes interconectados y versiones frecuentes, las degradaciones pueden ocurrir en cualquier momento. Y, si va a desarrollar una aplicación, los usuarios normalmente encuentran problemas que no se han detectado durante las pruebas. Debe tener conocimiento de estos problemas de inmediato y disponer de las herramientas de diagnóstico y solución de problemas.

[Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor) es una herramienta básica para la supervisión de servicios que se ejecutan en Azure. Proporciona datos a nivel de infraestructura sobre el rendimiento de un servicio y el entorno circundante. Si va a administrar todas las aplicaciones en Azure y debe decidir si quiere ampliar o reducir los recursos, Azure Monitor es el punto de partida.

También puede usar datos de supervisión para extraer conclusiones detalladas sobre la aplicación. Este conocimiento puede ayudarle a mejorar el rendimiento o mantenimiento de la aplicación, o a automatizar acciones que de lo contrario requerirían intervención manual.

Azure Monitor incluye los siguientes componentes.

### <a name="azure-activity-log"></a>Azure Activity Log

El [registro de actividad de Azure](/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs) proporciona información sobre las operaciones que se realizaron en los recursos de su suscripción. Antes, se conocía como "Registro de auditoría" o "Registro operativo", ya que notifica eventos del plano de control para las suscripciones.

### <a name="azure-diagnostic-logs"></a>Registros de diagnósticos de Azure

Un recurso emite [registros de diagnóstico de Azure](/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs) que proporcionan datos exhaustivos y frecuentes acerca del funcionamiento de ese recurso. El contenido de estos registros varía según el tipo de recurso.

Los registros del sistema de eventos de Windows son una categoría de registros de diagnóstico para VM. Los registros de BLOB, tabla y cola son categorías de registros de diagnóstico para cuentas de almacenamiento.

Lo registros de diagnóstico son distintos del [registro de actividad](/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs). El registro de actividad proporciona información sobre las operaciones que se realizaron en los recursos de su suscripción. Los registros de diagnóstico proporcionan conclusiones detalladas sobre las operaciones que el propio recurso realiza.

### <a name="metrics"></a>Métricas

Azure Monitor proporciona telemetría que le ofrece visibilidad sobre el rendimiento y el estado de las cargas de trabajo en Azure. El tipo de telemetría de datos de Azure más importante son las [métricas](/azure/monitoring-and-diagnostics/monitoring-overview-metrics) (también denominadas contadores de rendimiento) emitidas por la mayoría de los recursos de Azure. Azure Monitor proporciona varias maneras de configurar y usar estas métricas para supervisar y solucionar problemas.

### <a name="azure-diagnostics"></a>Diagnóstico de Azure

Azure Diagnostics habilita la recopilación de datos de diagnóstico en una aplicación implementada. Puede utilizar la extensión de Diagnostics desde diversos orígenes. Actualmente, se admiten [roles de servicio en la nube de Azure](/azure/vs-azure-tools-configure-roles-for-cloud-service), [máquinas virtuales de Azure](/azure/vs-azure-tools-configure-roles-for-cloud-service) que ejecutan Microsoft Windows y [Azure Service Fabric](/azure/monitoring-and-diagnostics/azure-diagnostics).

## <a name="azure-network-watcher"></a>Azure Network Watcher

Los clientes pueden crear una red integral en Azure mediante la orquestación y composición de varios recursos de red individuales, como, por ejemplo, redes virtuales, Azure ExpressRoute, Azure Application Gateway y equilibradores de carga. Se puede supervisar cada uno de los recursos de la red.

La red integral puede tener configuraciones complejas e interacciones entre recursos. El resultado es un escenario complejo que necesita supervisión basada en el escenario a través de [Azure Network Watcher](/azure/network-watcher/network-watcher-monitoring-overview).

Network Watcher simplifica la supervisión y diagnóstico de la red de Azure. Puede usar las herramientas de diagnóstico y visualización de Network Watcher para:

- Realizar capturas de paquetes remotos en una máquina virtual de Azure.
- Extraer conclusiones sobre el tráfico de la red mediante registros de flujo.
- Diagnosticar Azure VPN Gateway y conexiones.

En la actualidad, Network Watcher dispone de las siguientes funcionalidades:

- [Topología](/azure/network-watcher/network-watcher-topology-overview): proporciona una vista de las diversas interconexiones y asociaciones entre los recursos de red de un grupo de recursos.
- [Captura de paquetes variable](/azure/network-watcher/network-watcher-packet-capture-overview): captura datos de paquetes dentro y fuera de una máquina virtual. Las opciones de filtrado avanzadas y controles optimizados, con los que se pueden establecer límites de tiempo y de tamaño, proporcionan una gran versatilidad. Los datos de paquete se pueden almacenar en un almacén de blobs o en el disco local en formato .cap.
- [Comprobar el flujo de IP](/azure/network-watcher/network-watcher-ip-flow-verify-overview): comprueba si un paquete está permitido o no en función de los parámetros del paquete de 5 tuplas de información del flujo (IP de destino, IP de origen, puerto de destino, puerto de origen y protocolo). Si un grupo de seguridad deniega un paquete, se devuelve la regla y el grupo que lo deniegan.
- [Próximo salto](/azure/network-watcher/network-watcher-next-hop-overview): determina el próximo salto para los paquetes que se enrutan en el tejido de red de Azure, lo que le permite diagnosticar cualquier ruta definida por el usuario que se haya configurado incorrectamente.
- [Vista de grupo de seguridad](/azure/network-watcher/network-watcher-security-group-view-overview): Obtiene las reglas de seguridad eficaces que se aplican en una máquina virtual.
- [Registros de flujo de NSG para grupos de seguridad de red ](/azure/network-watcher/network-watcher-nsg-flow-logging-overview): Permiten capturar registros relacionados con el tráfico que las reglas de seguridad del grupo aprueban o deniegan. El flujo se define mediante una información de 5 tuplas: IP de origen, IP de destino, puerto de origen, puerto de destino y protocolo.
- [Solución de problemas de las conexiones y la puerta de enlace de Virtual Network](/azure/network-watcher/network-watcher-troubleshoot-manage-rest): Proporciona la capacidad para solucionar problemas con las puertas de enlace de red virtual y las conexiones.
- [Límites de suscripción de red](/azure/network-watcher/network-watcher-monitoring-overview): permite ver el uso de los recursos de la red en comparación con los límites.
- [Registros de diagnóstico](/azure/network-watcher/network-watcher-monitoring-overview): Proporciona un único panel para habilitar o deshabilitar los registros de diagnóstico para los recursos de red de un grupo de recursos.

Para más información, consulte [Configurar Network Watcher](/azure/network-watcher/network-watcher-create).

## <a name="cloud-service-provider-access-transparency"></a>Transparencia de acceso del proveedor de servicio en la nube

[Caja de seguridad del cliente de Microsoft Azure](customer-lockbox-overview.md) es un servicio integrado en Azure Portal que proporciona un control explícito en aquellos casos poco frecuentes en que el ingeniero de soporte técnico de Microsoft necesita acceso a sus datos para resolver un problema.
Son muy escasas las ocasiones en que un ingeniero de soporte técnico de Microsoft requiere permisos elevados para resolver un problema; por ejemplo, la depuración de un acceso remoto. En tales casos, los ingenieros de Microsoft usan el servicio de acceso just-in-time que proporciona autorización temporal y limitada con acceso restringido al servicio.  
Si bien Microsoft siempre ha obtenido el consentimiento del usuario para el acceso, la Caja de seguridad del cliente ahora la ofrece la posibilidad de revisar y aprobar o denegar tales solicitudes desde Azure Portal. Los ingenieros de soporte técnico de Microsoft no tendrán acceso hasta que se apruebe la solicitud.

## <a name="standardized-and-compliant-deployments"></a>Implementaciones estandarizadas y conformes

Los [planos técnicos de Azure](/azure/governance/blueprints/overview) permiten a los grupos de arquitectos de la nube y de TI central definir un conjunto repetible de recursos de Azure que implementa y cumple los estándares de la organización, sus requisitos y sus patrones.  
Esto hace posible que los equipos de DevOps compilen y pongan en funcionamiento rápidamente nuevos entornos y confíen en que los están generando con una infraestructura que mantiene el cumplimiento de la organización.
Los planos técnicos ofrecen una manera declarativa de organizar la implementación de varias plantillas de recursos y de otros artefactos, como son:

- Asignaciones de roles
- Asignaciones de directiva
- Plantillas del Administrador de recursos de Azure
- Grupos de recursos

## <a name="devops"></a>DevOps

Antes del desarrollo de aplicaciones de [Developer Operations (DevOps)](https://www.visualstudio.com/learn/what-is-devops/), los equipos eran los responsables de recopilar los requisitos de negocio para un programa de software y de escribir el código. A continuación, un equipo independiente de QA probaba el programa en un entorno de desarrollo aislado. Si se cumplían los requisitos, el equipo de QA liberaba el código de las operaciones para implementarlo. Los equipos de implementación se dividieron más en grupos, como redes y bases de datos. Cada vez que se pasaba un programa de software a un equipo independiente, se agregaban cuellos de botella.

DevOps permite a los equipos ofrecer soluciones más seguras y de mayor calidad, además de más rápidas y baratas. Los clientes esperan una experiencia dinámica y confiable al consumir servicios y software. Los equipos deben iterar rápidamente en actualizaciones de software y medir el impacto de las actualizaciones. Deben responder rápidamente con nuevas iteraciones de desarrollo para solucionar problemas o para proporcionar más valor.  

Las plataformas en la nube, como Microsoft Azure, han quitado los cuellos de botella tradicionales y ayudaron a homogeneizar la infraestructura. El software impera en cada negocio como factor y diferenciador claves en los resultados empresariales. Ninguna organización, desarrollador o trabajador de TI puede o debe evitar el movimiento de DevOps.

Los profesionales de DevOps consolidados adoptarán algunos de los siguientes procedimientos. Estos procedimientos [implican que personas](https://www.visualstudio.com/learn/what-is-devops-culture/) realicen estrategias basadas en escenarios empresariales. Las herramientas pueden ayudar a automatizar varios procedimientos.

- Las técnicas de [administración de proyectos y planificación ágiles](https://www.visualstudio.com/learn/what-is-agile/) se usan para planear y aislar el trabajo en sprints, administrar la capacidad del equipo y ayudar a los equipos a adaptarse rápidamente ante las cambiantes necesidades empresariales.
- [Control de versiones, normalmente con Git](https://www.visualstudio.com/learn/what-is-git/), permite que los equipos ubicados en cualquier lugar del mundo para compartir código fuente e integrarlo con herramientas de desarrollo de software para automatizar la canalización de versiones.
- La [integración continua](https://www.visualstudio.com/learn/what-is-continuous-integration/) impulsa la fusión en curso y la prueba de código, que permite la detección temprana de defectos.  Entre otras ventajas se incluye menos tiempo perdido en la lucha de problemas de fusión y comentarios rápidos para los equipos de desarrollo.
- La [entrega continua](https://www.visualstudio.com/learn/what-is-continuous-delivery/) de soluciones de software para los entornos de producción y prueba ayudan a las organizaciones a solucionar los problemas rápidamente y a responder a los requisitos empresariales en constante cambio.
- La [supervisión](https://www.visualstudio.com/learn/what-is-monitoring/) de aplicaciones en ejecución, incluidos los entornos de producción para el estado de la aplicación, así como el uso del cliente, ayudan a las organizaciones a plantear una hipótesis y validar o refutar estrategias rápidamente.  Los datos enriquecidos se capturan y almacenan en distintos formatos de registro.
- La [infraestructura como código (IaC)](https://www.visualstudio.com/learn/what-is-infrastructure-as-code/) es un procedimiento que permite la automatización y validación de la creación y destrucción de redes y máquinas virtuales para ayudar a proporcionar plataformas de hospedaje de aplicaciones estables y seguras.
- La arquitectura de [microservicios](https://www.visualstudio.com/learn/what-are-microservices/) se usa para aislar casos de uso empresariales en pequeños servicios reutilizables.  Esta arquitectura permite la escalabilidad y la eficacia.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre la solución Seguridad y auditoría, vea los artículos siguientes:

- [Seguridad y cumplimiento normativo](https://azure.microsoft.com/overview/trusted-cloud/)
- [Azure Security Center](/azure/security-center/security-center-intro)
- [Azure Monitor](/azure/azure-monitor/overview)
