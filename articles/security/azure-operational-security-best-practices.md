---
title: Prácticas recomendadas de seguridad para proteger sus activos, Microsoft Azure
description: En este artículo se proporciona un conjunto de procedimientos recomendados operativos para proteger los datos, aplicaciones y otros recursos en Azure.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/06/2019
ms.author: terrylan
ms.openlocfilehash: 0418d325f3b3719549181a48fc0432a677f695d5
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65795898"
---
# <a name="azure-operational-security-best-practices"></a>Procedimientos recomendados de seguridad operativa de Azure
En este artículo se proporciona un conjunto de procedimientos recomendados operativos para proteger los datos, aplicaciones y otros recursos en Azure.

Los procedimientos recomendados se basan en un consenso de opinión y son válidos para las funcionalidades y conjuntos de características actuales de la plataforma Azure. Las opiniones y las tecnologías cambian con el tiempo y en este artículo se actualiza de forma periódica para reflejar dichos cambios.

## <a name="define-and-deploy-strong-operational-security-practices"></a>Definir e implementar prácticas recomendadas de fuerte seguridad operativa
Por seguridad operativa de Azure, se entienden los servicios, los controles y las características disponibles para los usuarios para proteger sus datos, aplicaciones y otros recursos en Azure. La seguridad operativa de Azure se basa en un marco que incorpora el conocimiento adquirido a través de diversas funcionalidades exclusivas de Microsoft, incluido el [ciclo de vida de desarrollo de seguridad (SDL)](https://www.microsoft.com/sdl), el programa [Microsoft Security Response Center](https://www.microsoft.com/msrc?rtc=1) y un conocimiento en profundidad del panorama de amenazas de ciberseguridad.

## <a name="manage-and-monitor-user-passwords"></a>Administrar y supervisar las contraseñas de usuario
En la tabla siguiente se enumera algunos procedimientos recomendados relacionados con la administración de contraseñas de usuario:

**Procedimiento recomendado**: Asegúrese de que tener el nivel adecuado de protección mediante contraseña en la nube.   
**Detalles**: Siga las instrucciones de [orientación de la contraseña de Microsoft](https://www.microsoft.com/research/publication/password-guidance/), que tiene un ámbito a los usuarios de las plataformas de identidad de Microsoft (cuenta de Azure Active Directory, Active Directory y Microsoft).

**Procedimiento recomendado**: Supervisión de acciones sospechosas relacionadas con las cuentas de usuario.   
**Detalles**: Supervisar para [usuarios en riesgo](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk) y [inicios de sesión](../active-directory/reports-monitoring/concept-risk-events.md) mediante el uso de seguridad de Azure AD los informes.

**Procedimiento recomendado**: Detectar y corregir las contraseñas de alto riesgo automáticamente.   
**Detalles**: [Azure AD Identity Protection](../active-directory/identity-protection/overview.md) es una característica de Azure AD Premium P2 edition que le permite:

- Detectar posibles vulnerabilidades que afectan a las identidades de su organización
- Configurar respuestas automatizadas a acciones sospechosas detectadas que están relacionadas con las identidades de la organización
- Investigar incidentes sospechosos y tomar las medidas adecuadas para resolverlos

## <a name="receive-incident-notifications-from-microsoft"></a>Recibir notificaciones de incidentes de Microsoft
Asegúrese de que su equipo de operaciones de seguridad recibe notificaciones de incidentes de Azure de Microsoft. Una permite notificación de incidentes sabe de su equipo de seguridad ha puesto en peligro los recursos de Azure para poder responder a rápidamente y corregir posibles riesgos de seguridad.

En el portal de inscripción de Azure, puede asegurarse de información de contacto del administrador incluye detalles que notificar a las operaciones de seguridad. La información de los contactos es una dirección de correo electrónico y un número de teléfono.

## <a name="organize-azure-subscriptions-into-management-groups"></a>Organizar las suscripciones de Azure en grupos de administración
Si su organización tiene varias suscripciones, podría necesitar una manera de administrar el acceso, las directivas y el cumplimiento de esas suscripciones de forma eficaz. [Grupos de administración de Azure](../governance/management-groups/create.md) proporcionan un nivel de ámbito que está por encima de las suscripciones. Organizar las suscripciones en contenedores llamados grupos de administración y se aplican sus condiciones de gobierno a los grupos de administración. Todas las suscripciones dentro de un grupo de administración heredan automáticamente las condiciones que se aplican al grupo de administración.

Puede crear una estructura flexible de grupos de administración y las suscripciones en un directorio. Cada directorio tiene un grupo de administración de nivel superior único denominado grupo de administración raíz. Este grupo de administración raíz está integrado en la jerarquía de manera que contiene todos los grupos de administración y suscripciones. El grupo de administración raíz permite que las directivas globales y las asignaciones de RBAC para aplicarse en el nivel de directorio.

Estas son algunas prácticas recomendadas para el uso de grupos de administración:

**Procedimiento recomendado**: Asegúrese de que nuevas suscripciones aplican elementos de gobierno, como las directivas y permisos de medida que se agregan.   
**Detalles**: Use el grupo de administración raíz para asignar elementos de seguridad de toda la empresa que se aplican a todos los recursos de Azure. Las directivas y permisos son ejemplos de elementos.

**Procedimiento recomendado**: Alinear los niveles superiores de grupos de administración con la estrategia de segmentación para proporcionar un punto de control y la directiva de coherencia dentro de cada segmento.   
**Detalles**: Cree un grupo de administración única para cada segmento en el grupo de administración raíz. No se crean otros grupos de administración en el directorio raíz.

**Procedimiento recomendado**: Limitar la profundidad de grupo de administración para evitar la confusión que imposibilita las operaciones y seguridad.   
**Detalles**: Limitar la jerarquía de tres niveles, incluida la raíz.

**Procedimiento recomendado**: Seleccionar cuidadosamente qué elementos se aplican a toda la empresa con el grupo de administración raíz.   
**Detalles**: Asegúrese de elementos del grupo de administración raíz tienen un claro deben aplicarse a través de todos los recursos y que están bajo impacto.

Buenos candidatos son:

- Requisitos normativos que tienen un impacto de negocio claro (por ejemplo, restricciones relacionadas con la soberanía de datos)
- Requisitos con el potencial de casi cero negativo afectan a en las operaciones, como la directiva con efecto de auditoría o RBAC asignaciones de permisos que se han revisado con cuidado

**Procedimiento recomendado**: Planear y probar todos los cambios de toda la empresa en el grupo de administración raíz antes de aplicarlos (directiva, el modelo RBAC etc.).   
**Detalles**: Cambios en el grupo de administración raíz pueden afectar a todos los recursos en Azure. Aunque proporcionan una manera eficaz para garantizar la coherencia en toda la empresa, errores o uso incorrecto puede afectar negativamente a las operaciones de producción. Probar todos los cambios en el grupo de administración raíz en un laboratorio de pruebas o producción piloto.

## <a name="streamline-environment-creation-with-blueprints"></a>Simplificar la creación del entorno con planos técnicos
[Los planos Azure](../governance/blueprints/overview.md) servicio permite que los arquitectos de la nube y grupos de tecnología de información central definir un conjunto repetible de recursos de Azure que implementa y se adhiere patrones, estándares y requisitos de la organización. Planos técnicos de Azure permite a los equipos de desarrollo compilar rápidamente y crear nuevos entornos con un conjunto de componentes integrados y la confianza que va a crear esos entornos dentro de cumplimiento organizativo.

## <a name="monitor-storage-services-for-unexpected-changes-in-behavior"></a>Supervisar servicios de almacenamiento para detectar cambios inesperados de comportamiento
Diagnosticar y solucionar problemas en una aplicación distribuida hospedada en un entorno de nube puede ser más complicado que en los entornos tradicionales. Las aplicaciones se pueden implementar en una infraestructura PaaS o IaaS, en una ubicación local, en un dispositivo móvil o en varios de estos entornos combinados. El tráfico de red de la aplicación puede atravesar redes públicas y privadas, y la aplicación podría usar varias tecnologías de almacenamiento.

Debe supervisar continuamente los servicios de almacenamiento que usa la aplicación para detectar cualquier cambio inesperado de comportamiento (por ejemplo, tiempos de respuesta más lentos). Use los registros para recopilar datos más detallados y analizar un problema en profundidad. La información de diagnóstico obtenida de la supervisión y los registros le ayuda a determinar la causa raíz del problema detectado por la aplicación. Luego puede solucionar el problema y determinar los pasos adecuados para remediarlo.

[Azure Storage Analytics](../storage/storage-analytics.md) registra y proporciona datos de métricas de una cuenta de Azure Storage. Se recomienda usar estos datos para hacer un seguimiento de solicitudes, analizar tendencias de uso y diagnosticar problemas con la cuenta de almacenamiento.

## <a name="prevent-detect-and-respond-to-threats"></a>Prevención de las amenazas, detección y respuesta
[Azure Security Center](../security-center/security-center-intro.md) le ayuda a evitar, detectar y responder a amenazas, ya que proporciona mayor visibilidad (y controlar a través de) la seguridad de los recursos de Azure. Proporciona administración de directivas y supervisión de seguridad integradas en las suscripciones de Azure, ayuda a detectar las amenazas que podrían pasar desapercibidas y funciona con distintas soluciones de seguridad.

El nivel gratis de Security Center ofrece seguridad limitada para solo los recursos de Azure. El nivel Estándar amplía estas funcionalidades al entorno local y otras nubes. Estándar de Security Center le ayuda a encontrar y corregir las vulnerabilidades de seguridad, aplicar controles de acceso y de aplicación para bloquear actividades malintencionadas, detectar amenazas mediante análisis e inteligencia y responder rápidamente cuando esté siendo atacado. Dicho plan se puede probar de forma gratuita los primeros 60 días. Se recomienda [actualizar su suscripción de Azure a Security Center estándar](../security-center/security-center-get-started.md).

Use el centro de seguridad para obtener una visión central del estado de seguridad de todos los recursos de Azure. Compruebe que los controles de seguridad adecuados se han implementado y configurado correctamente, e identifique de un vistazo cualquier recurso que demande atención.

Security Center se integra también con [protección de amenazas avanzada de Windows Defender (ATP)](../security-center/security-center-wdatp.md), que proporciona capacidades completas de punto de conexión de detección y respuesta (EDR). Con la integración de ATP de Windows Defender, puede detectar anomalías. También puede detectar y responder a ataques avanzados en los puntos de conexión de servidor supervisados por Security Center.

Casi todas las organizaciones empresariales tienen un sistema de administración (SIEM) de eventos e información de seguridad para ayudar a identificar las amenazas emergentes mediante la consolidación de información de registro de recopilación de la señal diversos dispositivos. A continuación, se analizan los registros mediante un sistema de análisis de datos para ayudar a identificar lo que es "interesante" del ruido que es inevitable en todas las soluciones de análisis y recopilación de registros.

[Azure centinela](../sentinel/overview.md) es una solución de respuesta (eleva VERTIGINOSAMENTE) de orquestación automatizada de seguridad y administración de eventos e información de seguridad en la nube y nativo, escalable (SIEM). Sentinel Azure proporciona inteligencia de amenazas y análisis de seguridad inteligente a través de la detección de alertas, amenazas visibilidad, caza proactiva y respuesta automática de las amenazas.

Estos son algunos procedimientos recomendados para evitar, detectar y responder a amenazas:

**Procedimiento recomendado**: Aumentar la velocidad y escalabilidad de su solución SIEM mediante el uso de una solución de SIEM en la nube.   
**Detalles**: Investigar las características y capacidades de [Azure Sentinel](../sentinel/overview.md) y compararlos con las capacidades de lo que está actualmente mediante un entorno local. Considere la posibilidad de adoptar Azure Sentinel si cumple los requisitos de SIEM de su organización.

**Procedimiento recomendado**: Encuentre las vulnerabilidades de seguridad más graves, por lo que puede dar prioridad a la investigación.   
**Detalles**: Revise su [puntuación segura de Azure](../security-center/security-center-secure-score.md) para ver las recomendaciones resultantes de las directivas de Azure y las iniciativas integradas en Azure Security Center. Estas recomendaciones ayudan a abordar los riesgos principales como las actualizaciones de seguridad, protección de extremos, cifrado, las configuraciones de seguridad, WAF que faltan, conectado a internet de las máquinas virtuales y muchos más.

La puntuación segura, que se basa en el Centro para los controles de seguridad de Internet (CIS), le permite realizar pruebas comparativas de seguridad de Azure de su organización contra orígenes externos. Validación externa ayuda a validar y enriquecer la estrategia de seguridad de su equipo.

**Procedimiento recomendado**: Supervisar la postura de seguridad de máquinas, redes, almacenamiento y servicios de datos y las aplicaciones detectar y dar prioridad a posibles problemas de seguridad.  
**Detalles**: Siga el [recomendaciones de seguridad](../security-center/security-center-recommendations.md) iniciar Security Center, con los elementos de prioridad más alta.

**Procedimiento recomendado**: Integrar las alertas de Security Center en su solución de administración (SIEM) de eventos e información de seguridad.   
**Detalles**: Mayoría de las organizaciones con una solución SIEM utilizarlo como un centro de intercambio para las alertas de seguridad que requieren una respuesta de analista. Eventos procesados generados por Security Center se publican en el registro de actividad de Azure, uno de los registros disponibles a través de Azure Monitor. Azure Monitor ofrece una canalización consolidada para el enrutamiento de cualquiera de los datos supervisados en una herramienta SIEM. Consulte [integrar soluciones de seguridad en Security Center](../security-center/security-center-partner-integration.md#exporting-data-to-a-siem) para obtener instrucciones. Si usas Centinela de Azure, consulte [Connect Azure Security Center](../sentinel/connect-azure-security-center.md).

**Procedimiento recomendado**: Integrar Azure los registros de SIEM.   
**Detalles**: Use [Azure Monitor para recopilar y exportar datos](../azure-monitor/overview.md#integrate-and-export-data). Esta práctica es fundamental para habilitar la investigación de incidentes de seguridad y retención de registro en línea es limitada. Si usas Centinela de Azure, consulte [conectar orígenes de datos](../sentinel/connect-data-sources.md).

**Procedimiento recomendado**: Acelerar la investigación y los procesos de búsqueda y reducir los falsos positivos mediante la integración de capacidades de punto de conexión de detección y respuesta (EDR) en la investigación de ataque.   
**Detalles**: [Habilitar la integración de ATP de Windows Defender](../security-center/security-center-wdatp.md#enable-windows-defender-atp-integration) a través de la directiva de seguridad de Security Center. Considere el uso de Azure Centinela para búsqueda de amenazas y respuesta a incidentes.

## <a name="monitor-end-to-end-scenario-based-network-monitoring"></a>Supervisión de redes de un extremo a otro basada en escenarios
Los clientes crean una red de un extremo a otro en Azure mediante la combinación de recursos de red como una red virtual, ExpressRoute, Application Gateway y equilibradores de carga. Se puede supervisar cada uno de los recursos de la red.

[Azure Network Watcher](../network-watcher/network-watcher-monitoring-overview.md) es un servicio regional. Use sus herramientas de diagnóstico y visualización para supervisar y diagnosticar problemas en un nivel de escenario de red en, hacia y desde Azure.

A continuación se ofrecen procedimientos recomendados para la supervisión de redes y se indican las herramientas disponibles.

**Procedimiento recomendado**: Automatización de la supervisión de la red remota con captura de paquetes.  
**Detalles**: supervise y diagnostique problemas de red sin iniciar sesión en las máquinas virtuales mediante Network Watcher. Desencadene la [captura de paquetes](../network-watcher/network-watcher-alert-triggered-packet-capture.md) estableciendo alertas y obtenga acceso a información de rendimiento en tiempo real en el ámbito de paquete. Cuando vea un problema, podrá investigar en detalle para mejorar los diagnósticos.

**Procedimiento recomendado**: Extraer conclusiones sobre el tráfico de la red mediante registros de flujo.  
**Detalles**: conozca al detalle los patrones de tráfico de red mediante los [registros de flujo del grupo de seguridad de red](../network-watcher/network-watcher-nsg-flow-logging-overview.md). La información de los registros de flujo le ayuda a recopilar datos para el cumplimiento, la auditoría y la supervisión del perfil de seguridad de red.

**Procedimiento recomendado**: diagnóstico de problemas de conectividad VPN.  
**Detalles**: use Network Watcher para [diagnosticar los problemas más comunes de conexión y VPN Gateway](../network-watcher/network-watcher-diagnose-on-premises-connectivity.md). No solo puede identificar el problema, sino también usar registros detallados para investigar más.

## <a name="secure-deployment-by-using-proven-devops-tools"></a>Implementación segura mediante herramientas de DevOps comprobadas
Use los siguientes procedimientos recomendados de DevOps para garantizar que la empresa y los equipos sean productivos y eficientes.

**Procedimiento recomendado**: automatizar la compilación e implementación de servicios.  
**Detalles**: [Infraestructura como código](https://docs.microsoft.com/azure/devops/learn/what-is-infrastructure-as-code) es un conjunto de técnicas y procedimientos que ayudan a los profesionales de TI a eliminar la carga que supone la compilación y administración cotidianas de una infraestructura modular. Permite a los profesionales de TI compilar y mantener sus entornos de servidores modernos de forma similar a como los desarrolladores de software compilan y mantienen el código de las aplicaciones.

Puede usar [Azure Resource Manager](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/) para aprovisionar las aplicaciones mediante una plantilla declarativa. En una plantilla, puede implementar varios servicios junto con sus dependencias. Use la misma plantilla para implementar la aplicación de forma repetida durante cada fase de su ciclo de vida.

**Procedimiento recomendado**: compilar e implementar automáticamente en aplicaciones web o servicios en la nube de Azure.  
**Detalles**: Puede configurar los proyectos de DevOps de Azure para [Compile e implemente automáticamente](https://docs.microsoft.com/azure/devops/pipelines/index?azure-devops) a Azure web apps o cloud services. Azure DevOps implementa automáticamente los archivos binarios después de realizar una compilación en Azure tras cada comprobación de código. El proceso de compilación del paquete es equivalente al comando Package de Visual Studio y los pasos de publicación son equivalentes al comando Publish de Visual Studio.

**Procedimiento recomendado**: Automatizar la administración de versiones.  
**Detalles**: [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/index?azure-devops) es una solución para automatizar la implementación en varias fases y la administración del proceso de publicación. Cree canalizaciones de implementación continua y administrada con el fin de que el lanzamiento sea rápido, sencillo y frecuente. Con Azure Pipelines, puede automatizar el proceso de publicación y tener flujos de trabajo de aprobación predefinidos. Realice la implementación de forma local y en la nube, amplíela y personalícela según sea necesario.

**Procedimiento recomendado**: Compruebe el rendimiento de su aplicación antes de iniciarla o de implementar actualizaciones en la producción.  
**Detalles**: Ejecutar en la nube [las pruebas de carga](/azure/devops/test/load-test/overview#alternatives) para:

- Detectar problemas de rendimiento en la aplicación.
- Mejorar la calidad de implementación.
- Asegurarse de que la aplicación siempre está disponible.
- Asegurarse de que la aplicación puede controlar el tráfico de la siguiente campaña de marketing o el siguiente lanzamiento.

[Apache JMeter](https://jmeter.apache.org/) es una herramienta gratuita de código abierto populares con una sólida comunidad de seguridad.

**Procedimiento recomendado**: supervisar el rendimiento de las aplicaciones.  
**Detalles**: [Azure Application Insights](../azure-monitor/app/app-insights-overview.md) es un servicio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas. Use Application Insights para supervisar la aplicación web en vivo. Se detectan automáticamente las anomalías de rendimiento. Incluye herramientas de análisis que le ayudan a diagnosticar problemas y a comprender lo que hacen realmente los usuarios con la aplicación. Está diseñado para ayudarle a mejorar continuamente el rendimiento y la facilidad de uso.

## <a name="mitigate-and-protect-against-ddos"></a>Mitigar y proteger frente a DDoS
La denegación de servicio distribuido (DDoS) es un tipo de ataque que intenta agotar los recursos de la aplicación. El objetivo es afectar a la disponibilidad de la aplicación y a su capacidad para administrar solicitudes legítimas. Estos ataques son cada vez más sofisticados y tienen un mayor tamaño e impacto. Pueden ir dirigidos a cualquier punto de conexión que sea públicamente accesible a través de Internet.

Diseñar y compilar para la resistencia frente a DDoS requiere planear y diseñar para una serie de modos de error. Estos son los procedimientos recomendados para compilar servicios resistentes a DDoS en Azure.

**Procedimiento recomendado**: Asegúrese de que la seguridad es prioritaria durante todo el ciclo de vida de una aplicación, desde su diseño e implementación hasta la implementación y las operaciones. Las aplicaciones pueden tener errores que permiten que un volumen relativamente bajo de solicitudes usen muchos recursos y produzcan una interrupción del servicio.  
**Detalles**: Para proteger un servicio que se ejecuta en Microsoft Azure, debe conocer bien la arquitectura de su aplicación y centrarse en los [cinco pilares de la calidad del software](https://docs.microsoft.com/azure/architecture/guide/pillars). Debe conocer los volúmenes de tráfico típicos, el modelo de conectividad entre la aplicación y otras aplicaciones, y los puntos de conexión del servicio expuestos a la red pública de Internet.

Es de vital importancia garantizar que una aplicación sea lo suficientemente resistente para tratar con un ataque de denegación de servicio dirigido a la propia aplicación. La seguridad y la privacidad están integradas en la plataforma Azure, comenzando por el [ciclo de vida del desarrollo de la seguridad (SDL)](https://www.microsoft.com/sdl). El SDL aborda la seguridad en cada fase de desarrollo y se asegura de que Azure se actualice continuamente para que sea aún más seguro.

**Procedimiento recomendado**: Debe diseñar sus aplicaciones de modo que se puedan [escalar horizontalmente](https://docs.microsoft.com/azure/architecture/guide/design-principles/scale-out) para satisfacer la demanda de una carga mayor, específicamente en caso de un ataque de DDoS. Si la aplicación depende de una única instancia de un servicio, crea un único punto de error. El aprovisionamiento de varias instancias hace que el sistema sea más resistente y más escalable.  
**Detalles**: Para [Azure App Service](../app-service/app-service-value-prop-what-is.md), seleccione un [Plan de App Service](../app-service/overview-hosting-plans.md) que ofrezca varias instancias.

Para Azure Cloud Services, configure cada uno de los roles para utilizar [varias instancias](../cloud-services/cloud-services-choose-me.md).

En el caso de [Azure Virtual Machines](../virtual-machines/windows/overview.md), asegúrese de que la arquitectura de las máquinas virtuales incluya más de una máquina virtual y de que cada una de ellas se incluya en un [conjunto de disponibilidad](../virtual-machines/virtual-machines-windows-manage-availability.md). Se recomienda usar conjuntos de escalado de máquinas virtuales para contar con funcionalidades de escalado automático.

**Procedimiento recomendado**: Disponer en niveles la defensa de la seguridad en una aplicación reduce las probabilidades de éxito de un ataque. Implemente diseños seguros para las aplicaciones con las funcionalidades integradas de la plataforma Azure.  
**Detalles**: el riesgo de ataque aumenta con el tamaño (área expuesta) de la aplicación. Puede reducir el área expuesta mediante la creación de listas blancas para cerrar el espacio de direcciones IP expuesto y los puertos de escucha que no sean necesarios en los equilibradores de carga ([Azure Load Balancer](../load-balancer/load-balancer-get-started-internet-portal.md) y [Azure Application Gateway](../application-gateway/application-gateway-create-probe-portal.md)).

Los [grupos de seguridad de red](../virtual-network/security-overview.md) constituyen otra manera de reducir el área expuesta a ataques. Puede usar [etiquetas de servicio](../virtual-network/security-overview.md#service-tags) y [grupos de seguridad de la aplicación](../virtual-network/security-overview.md#application-security-groups) para minimizar la complejidad de la creación de reglas de seguridad y configurar la seguridad de la red como una extensión natural de la estructura de una aplicación.

Debe implementar los servicios de Azure en una [red virtual](../virtual-network/virtual-networks-overview.md) siempre que sea posible. Este procedimiento permite que los recursos del servicio se comuniquen mediante direcciones IP privadas. De forma predeterminada, el tráfico de los servicios Azure desde una red virtual usa direcciones IP públicas como direcciones IP de origen.

Con los [puntos de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md), el tráfico del servicio cambia para usar direcciones privadas de red virtual como direcciones IP de origen al acceder al servicio de Azure desde una red virtual.

Con frecuencia vemos ataques a los recursos locales de un cliente, además de a los recursos en Azure. Si conecta un entorno local a Azure, minimice la exposición de los recursos locales a la red pública de Internet.

Azure tiene dos [ofertas de servicio](../virtual-network/ddos-protection-overview.md) de DDoS que proporcionan protección frente a ataques de red:

- La protección básica se integra en Azure de forma predeterminada sin costo adicional. El tamaño y la capacidad de la red de implementación global de Azure proporciona una defensa contra los ataques al nivel de red más comunes mediante la supervisión constante del tráfico y la mitigación en tiempo real. La protección básica no requiere ningún cambio de configuración ni de aplicación y ayuda a proteger todos los servicios de Azure, incluidos servicios PaaS como Azure DNS.
- La protección estándar proporciona funcionalidades avanzadas de mitigación de DDoS frente a ataques de red. Se ajusta automáticamente para proteger los recursos específicos de Azure. La protección se puede habilitar fácilmente durante la creación de redes virtuales. También puede realizarse después de la creación y no requiere ningún cambio de aplicación o recurso.

## <a name="enable-azure-policy"></a>Habilitar directiva de Azure
[Directiva de Azure](../governance/policy/overview.md) es un servicio de Azure que usó para crear, asignar y administrar directivas. Estas directivas aplican las reglas y los efectos sobre los recursos, por lo que estos sigan siendo compatibles con los estándares corporativos y los acuerdos de nivel de servicio. Azure Policy satisface esta necesidad mediante la evaluación de los recursos que incumplen las directivas asignadas.

Habilitar directiva de Azure para supervisar y aplicar la directiva escrita de su organización. Esto garantiza el cumplimiento de su empresa o requisitos de seguridad normativos administrando de forma centralizada las directivas de seguridad a través de las cargas de trabajo de nube híbrida. Obtenga información sobre cómo [crear y administrar directivas para exigir el cumplimiento](../governance/policy/tutorials/create-and-manage.md). Consulte [estructura de definición de directiva de Azure](../governance/policy/concepts/definition-structure.md) para obtener información general de los elementos de una directiva.

Estos son algunos procedimientos recomendados de seguridad para hacer seguimiento tras adoptar Azure Policy:

**Procedimiento recomendado**: Directiva admite varios tipos de efectos. Puede leer sobre ellos en [estructura de definición de directiva de Azure](../governance/policy/concepts/definition-structure.md#policy-rule). Operaciones de negocio pueden verse afectadas negativamente por la **denegar** efecto y el **corregir** efecto, así que comience con la **auditar** efecto para limitar el riesgo de impacto negativo de Directiva.   
**Detalles**: [Iniciar las implementaciones de directiva en modo auditoría](../governance/policy/concepts/definition-structure.md#policy-rule) y, a continuación, avanzar posteriormente a **denegar** o **corregir**. Prueba y revise los resultados del efecto de auditoría antes de pasar a **denegar** o **corregir**.

Para obtener más información, consulte [crear y administrar directivas para exigir el cumplimiento](../governance/policy/tutorials/create-and-manage.md).

**Procedimiento recomendado**: Identificar los roles responsables de supervisar las infracciones de directivas y garantizar la acción correctora adecuada se realiza rápidamente.   
**Detalles**: Tiene la compatibilidad de monitor de rol asignado a través de la [portal Azure](../governance/policy/how-to/get-compliance-data.md#portal) o a través de la [línea de comandos](../governance/policy/how-to/get-compliance-data.md#command-line).

**Procedimiento recomendado**: Directiva de Azure es una representación técnica de directivas escritas de la organización. Todas las directivas de Azure se asignan a las directivas organizativas para reducir la confusión y aumentar la coherencia.   
**Detalles**: Asignación de documentos en la documentación de su organización o en la directiva de Azure propio agregando una referencia a la directiva organizativa en Azure [descripción de la directiva](../governance/policy/concepts/definition-structure.md#display-name-and-description) o la directiva de Azure [iniciativa](../governance/policy/concepts/definition-structure.md#initiatives) Descripción.

## <a name="monitor-azure-ad-risk-reports"></a>Informes de riesgo de Monitor de Azure AD
La mayoría de las infracciones de seguridad tienen lugar cuando los atacantes obtienen acceso a un entorno mediante el robo de identidad de un usuario. Descubrir las identidades en peligro no es tarea fácil. Azure AD emplea algoritmos y heurística de aprendizaje automático adaptable para detectar acciones sospechosas que están relacionadas con las cuentas de usuario. Cada acción sospechosa detectada se almacena en un registro llamado [evento de riesgo](../active-directory/reports-monitoring/concept-risk-events.md). Se registran los eventos de riesgo de seguridad de Azure AD los informes. Para obtener más información, lea sobre la [informe usuarios en riesgo seguridad](../active-directory/reports-monitoring/concept-user-at-risk.md) y [informes de seguridad de inicios de sesión](../active-directory/reports-monitoring/concept-risky-sign-ins.md).

## <a name="next-steps"></a>Pasos siguientes
Vea [Patrones y procedimientos recomendados de seguridad en Azure](security-best-practices-and-patterns.md) para obtener más procedimientos recomendados de seguridad que pueda aplicar cuando diseñe, implemente y administre las soluciones en la nube mediante Azure.

En los siguientes recursos se ofrece más información general sobre la seguridad de Azure y los servicios de Microsoft relacionados:
* [Blog del equipo de seguridad de Azure](https://blogs.msdn.microsoft.com/azuresecurity/): ofrece información actualizada sobre lo último en seguridad de Azure
* [Microsoft Security Response Center](https://technet.microsoft.com/library/dn440717.aspx): aquí podrá notificar vulnerabilidades de seguridad de Microsoft, incluidos problemas con Azure, o también mediante correo electrónico a secure@microsoft.com.
