---
title: Procedimientos recomendados de seguridad para los recursos de Azure
titleSuffix: Azure security
description: En este artículo se proporciona un conjunto de procedimientos recomendados operativos para proteger los datos, aplicaciones y otros recursos en Azure.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: tomsh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/06/2019
ms.author: terrylan
ms.openlocfilehash: 496d99b3d871c66e2557e1f384bb4480cd8b0831
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423157"
---
# <a name="azure-operational-security-best-practices"></a>Procedimientos recomendados de seguridad operativa de Azure
En este artículo se proporciona un conjunto de procedimientos recomendados operativos para proteger los datos, aplicaciones y otros recursos en Azure.

Los procedimientos recomendados se basan en un consenso de opinión y son válidos para las funcionalidades y conjuntos de características actuales de la plataforma Azure. Como las opiniones y las tecnologías cambian con el tiempo, este artículo se actualiza de forma periódica para reflejar dichos cambios.

## <a name="define-and-deploy-strong-operational-security-practices"></a>Definición e implementación de procedimientos de seguridad operativa exhaustivos
Por seguridad operativa de Azure, se entienden los servicios, los controles y las características disponibles para los usuarios para proteger sus datos, aplicaciones y otros recursos en Azure. La seguridad operativa de Azure se basa en un marco que incorpora el conocimiento adquirido a través de diversas funcionalidades exclusivas de Microsoft, incluido el [ciclo de vida de desarrollo de seguridad (SDL)](https://www.microsoft.com/sdl), el programa [Microsoft Security Response Center](https://www.microsoft.com/msrc?rtc=1) y un conocimiento en profundidad del panorama de amenazas de ciberseguridad.

## <a name="manage-and-monitor-user-passwords"></a>Administración y supervisión de contraseñas de usuario
En la tabla siguiente se enumeran varios procedimientos recomendados relacionados con la administración de contraseñas de usuario:

**Procedimiento recomendado**: Asegúrese de que tiene el nivel adecuado de protección de contraseñas en la nube.   
**Detalles**: Siga las instrucciones de [Microsoft Password Guidance](https://www.microsoft.com/research/publication/password-guidance/) (Instrucciones para contraseñas de Microsoft), dirigidas a los usuarios de las plataformas de identidad de Microsoft (Azure Active Directory, Active Directory y cuenta de Microsoft).

**Procedimiento recomendado**: Supervise acciones sospechosas relacionadas con las cuentas de usuario.   
**Detalles**: Supervise [usuarios en riesgo](/azure/active-directory/reports-monitoring/concept-user-at-risk) e [inicios de sesión en riesgo](../../active-directory/reports-monitoring/concept-risk-events.md) mediante los informes de seguridad de Azure AD.

**Procedimiento recomendado**: Detectar y corregir de forma automática las contraseñas de alto riesgo.   
**Detalles**: [Azure AD Identity Protection](/azure/active-directory/identity-protection/overview) es una característica de la edición Azure AD Premium P2 que permite:

- Detectar posibles vulnerabilidades que afectan a las identidades de la organización
- Configurar respuestas automatizadas a acciones sospechosas detectadas que están relacionadas con las identidades de la organización
- Investigar incidentes sospechosos y tomar las medidas adecuadas para resolverlos

## <a name="receive-incident-notifications-from-microsoft"></a>Recepción de notificaciones de incidentes de Microsoft
Asegúrese de que el equipo de operaciones de seguridad recibe notificaciones de incidentes de Azure de Microsoft. Una notificación de incidentes permite al equipo de seguridad saber que hay recursos de Azure en peligro, para que puedan responder rápidamente y corregir posibles riesgos de seguridad.

En el portal de inscripción de Azure, puede asegurarse de que la información de contacto del administrador incluye detalles que notifican operaciones de seguridad. La información de los contactos es una dirección de correo electrónico y un número de teléfono.

## <a name="organize-azure-subscriptions-into-management-groups"></a>Organización de las suscripciones de Azure en grupos de administración
Si su organización tiene varias suscripciones, podría necesitar una manera de administrar el acceso, las directivas y el cumplimiento de esas suscripciones de forma eficaz. Los [grupos de administración de Azure](/azure/governance/management-groups/create) proporcionan un nivel de ámbito por encima de las suscripciones. Las suscripciones se organizan en contenedores llamados grupos de administración y las condiciones de gobernanza se aplican a los grupos de administración. Todas las suscripciones dentro de un grupo de administración heredan automáticamente las condiciones que se aplican al grupo de administración.

Puede crear una estructura flexible de grupos de administración y suscripciones en un directorio. A cada directorio se le asigna un único grupo de administración de nivel superior denominado grupo de administración raíz. Este grupo de administración raíz está integrado en la jerarquía de manera que contiene todos los grupos de administración y suscripciones. Este grupo de administración raíz permite que las directivas globales y las asignaciones de roles de Azure se apliquen en el nivel de directorio.

Estos son algunos procedimientos recomendados para el uso de grupos de administración:

**Procedimiento recomendado**: Asegúrese de que las suscripciones nuevas aplican los elementos de gobernanza, como directivas y permisos, a medida que se agregan.   
**Detalles**: Use el grupo de administración raíz para asignar elementos de seguridad de toda la empresa que se apliquen a todos los recursos de Azure. Las directivas y los permisos son ejemplos de elementos.

**Procedimiento recomendado**: Alinee los niveles superiores de los grupos de administración con la estrategia de segmentación para proporcionar un punto de control y directivas coherentes dentro de cada segmento.   
**Detalles**: Cree un único grupo de administración para cada segmento del grupo de administración raíz. No cree otros grupos de administración en el directorio raíz.

**Procedimiento recomendado**: Limite la profundidad del grupo de administración para evitar la confusión que imposibilita las operaciones y la seguridad.   
**Detalles**: Limite la jerarquía a tres niveles, incluido el nivel raíz.

**Procedimiento recomendado**: Seleccione cuidadosamente qué elementos se aplican a toda la empresa con el grupo de administración raíz.   
**Detalles**: Asegúrese de que la necesidad de aplicar los elementos del grupo de administración raíz en todos los recursos sea evidente y de que su impacto sea bajo.

Entre los candidatos adecuados destacan los siguientes:

- Requisitos normativos que tengan un impacto empresarial claro (por ejemplo, restricciones relacionadas con la soberanía de datos)
- Requisitos con un posible efecto negativo casi nulo en las operaciones, como directivas con efecto de auditoría o asignaciones de permisos RBAC que se hayan revisado con cuidado

**Procedimiento recomendado**: Planifique y pruebe con atención todos los cambios en toda la empresa en el grupo de administración raíz antes de aplicarlos (directiva, modelo RBAC, etc.).   
**Detalles**: Los cambios en el grupo de administración raíz pueden afectar a todos los recursos en Azure. Aunque proporcionan una manera eficaz de garantizar la coherencia en toda la empresa, los errores o un uso incorrecto pueden afectar negativamente a las operaciones de producción. Pruebe todos los cambios en el grupo de administración raíz en un laboratorio de pruebas o piloto de producción.

## <a name="streamline-environment-creation-with-blueprints"></a>Optimización de la creación de entornos con Blueprints
El servicio [Azure Blueprints](/azure/governance/blueprints/overview) permite a los arquitectos de nube y grupos de TI central definir un conjunto repetible de recursos de Azure que implementa y cumple los estándares, patrones y requisitos de la organización. Azure Blueprints permite a los equipos de desarrollo aprovisionar y crear rápidamente entornos con un conjunto de componentes integrados, con la confianza de que se crean de acuerdo a la normativa de la organización.

## <a name="monitor-storage-services-for-unexpected-changes-in-behavior"></a>Supervisar servicios de almacenamiento para detectar cambios inesperados de comportamiento
Diagnosticar y solucionar problemas en una aplicación distribuida hospedada en un entorno de nube puede ser más complicado que en los entornos tradicionales. Las aplicaciones se pueden implementar en una infraestructura PaaS o IaaS, en una ubicación local, en un dispositivo móvil o en varios de estos entornos combinados. El tráfico de red de la aplicación puede atravesar redes públicas y privadas, y la aplicación podría usar varias tecnologías de almacenamiento.

Debe supervisar continuamente los servicios de almacenamiento que usa la aplicación para detectar cualquier cambio inesperado de comportamiento (por ejemplo, tiempos de respuesta más lentos). Use los registros para recopilar datos más detallados y analizar un problema en profundidad. La información de diagnóstico obtenida de la supervisión y los registros le ayuda a determinar la causa raíz del problema detectado por la aplicación. Luego puede solucionar el problema y determinar los pasos adecuados para remediarlo.

[Azure Storage Analytics](../../storage/common/storage-analytics.md) registra y proporciona datos de métricas de una cuenta de Azure Storage. Se recomienda usar estos datos para hacer un seguimiento de solicitudes, analizar tendencias de uso y diagnosticar problemas con la cuenta de almacenamiento.

## <a name="prevent-detect-and-respond-to-threats"></a>Prevención de las amenazas, detección y respuesta
Para ayudar a evitar, detectar y responder a amenazas, [Azure Security Center](../../security-center/security-center-intro.md) proporciona más visibilidad y control sobre la seguridad de los recursos de Azure. Proporciona administración de directivas y supervisión de seguridad integradas en las suscripciones de Azure, ayuda a detectar las amenazas que podrían pasar desapercibidas y funciona con distintas soluciones de seguridad.

El nivel Gratis de Security Center ofrece seguridad limitada solo para los recursos de Azure. El nivel Estándar amplía estas funcionalidades al entorno local y otras nubes. El nivel Estándar de Security Center ayuda a encontrar y corregir vulnerabilidades de seguridad, aplicar controles de acceso y de aplicación para bloquear actividades malintencionadas, detectar amenazas mediante análisis e inteligencia, y responder rápidamente en caso de ataque. Dicho plan se puede probar de forma gratuita los primeros 60 días. Se recomienda [actualizar la suscripción de Azure al nivel Estándar de Security Center](../../security-center/security-center-get-started.md).

Use Security Center para obtener una vista centralizada del estado de seguridad de todos los recursos de Azure. Compruebe que los controles de seguridad adecuados se han implementado y configurado correctamente, e identifique de un vistazo cualquier recurso que demande atención.

Security Center también se integra con [Advanced Threat Protection (ATP) de Microsoft Defender](../../security-center/security-center-wdatp.md), que proporciona funcionalidades completas de detección y respuesta (EDR) de puntos de conexión. Gracias a la integración de ATP de Microsoft Defender, podrá detectar anomalías. También puede detectar y responder a ataques avanzados en los puntos de conexión de servidor supervisados por Security Center.

Casi todas las organizaciones empresariales tienen un sistema de administración de eventos e información de seguridad (SIEM) para facilitar la identificación de las amenazas emergentes mediante la consolidación de la información de registro de distintos dispositivos de recopilación de señales. Después, un sistema de análisis de datos analiza los registros para ayudar a identificar lo que es "interesante" del ruido que es inevitable en todas las soluciones de análisis y recopilación de registros.

[Azure Sentinel](/azure/sentinel/overview) es una solución de administración de eventos e información de seguridad (SIEM) y respuesta automatizada de orquestación de seguridad (SOAR) que es escalable y nativa de la nube. Azure Sentinel ofrece análisis de seguridad inteligentes e inteligencia frente a amenazas a través de la detección de alertas, la visibilidad de amenazas, la búsqueda proactiva y la respuesta automatizada antes las amenazas.

Estos son algunos procedimientos recomendados para evitar, detectar y responder a las amenazas:

**Procedimiento recomendado**: Aumente la velocidad y escalabilidad de la solución SIEM mediante SIEM basado en la nube.   
**Detalles**: Investigue las características y funciones de [Azure Sentinel](/azure/sentinel/overview) y compárelas con las de lo que use actualmente en el entorno local. Considere la posibilidad de adoptar Azure Sentinel si cumple los requisitos de SIEM de la organización.

**Procedimiento recomendado**: Identifique las vulnerabilidades de seguridad más graves para poder clasificar por orden de prioridad la investigación.   
**Detalles**: Revise la [puntuación segura de Azure](../../security-center/secure-score-security-controls.md) para ver las recomendaciones resultantes de las iniciativas y directivas de Azure integradas en Azure Security Center. Estas recomendaciones ayudan a abordar los riesgos más graves como actualizaciones de seguridad, protección de puntos de conexión, cifrado, configuraciones de seguridad, ausencia de WAF, máquinas virtuales conectadas a Internet y muchos más.

La puntuación segura, que se basa en controles del Centro de seguridad de Internet (CIS), permite realizar pruebas comparativas de la seguridad de Azure de la organización sobre orígenes externos. La validación externa ayuda a validar y enriquecer la estrategia de seguridad del equipo.

**Procedimiento recomendado**: Supervise la posición de seguridad de equipos, redes, almacenamiento y servicios de datos y aplicaciones para detectar y clasificar por orden de prioridad los posibles problemas de seguridad.  
**Detalles**: Siga las [recomendaciones de seguridad](../../security-center/security-center-recommendations.md) de Security Center, empezando con los elementos de mayor prioridad.

**Procedimiento recomendado**: Integre las alertas de Security Center en la solución de administración de eventos e información de seguridad (SIEM).   
**Detalles**: La mayoría de las organizaciones con una solución SIEM la usan como un centro de enrutamiento para las alertas de seguridad que requieren la respuesta de un analista. Los eventos procesados generados por Azure Security Center se publican en el Registro de actividad de Azure, uno de los registros disponibles a través de Azure Monitor. Azure Monitor ofrece una canalización consolidada para el enrutamiento de cualquiera de los datos supervisados en una herramienta SIEM. Consulte [Exportación de alertas y recomendaciones de seguridad](../../security-center/continuous-export.md#configuring-siem-integration-via-azure-event-hubs) para obtener instrucciones. Si usa Azure Sentinel, vea [Conexión de Azure Security Center](../../sentinel/connect-azure-security-center.md).

**Procedimiento recomendado**: Integre los registros de Azure con la solución SIEM.   
**Detalles**: Use [Azure Monitor para recopilar y exportar datos](/azure/azure-monitor/overview#integrate-and-export-data). Este procedimiento es fundamental para habilitar la investigación de incidentes de seguridad, y la retención de registro en línea es limitada. Si usa Azure Sentinel, vea [Conexión de orígenes de datos](../../sentinel/connect-data-sources.md).

**Procedimiento recomendado**: Acelere los procesos de investigación y búsqueda, y reduzca los falsos positivos mediante la integración de funciones de detección y respuesta (EDR) de puntos de conexión en la investigación del ataque.   
**Detalles**: [Habilite la integración de ATP de Microsoft Defender](../../security-center/security-center-wdatp.md#enable-microsoft-defender-atp-integration) mediante la directiva de seguridad de Security Center. Considere la posibilidad de usar Azure Sentinel para la búsqueda de amenazas y la respuesta a los incidentes.

## <a name="monitor-end-to-end-scenario-based-network-monitoring"></a>Supervisión de redes de un extremo a otro basada en escenarios
Los clientes crean una red de un extremo a otro en Azure mediante la combinación de recursos de red como una red virtual, ExpressRoute, Application Gateway y equilibradores de carga. Se puede supervisar cada uno de los recursos de la red.

[Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md) es un servicio regional. Use sus herramientas de diagnóstico y visualización para supervisar y diagnosticar problemas en un nivel de escenario de red en, hacia y desde Azure.

A continuación se ofrecen procedimientos recomendados para la supervisión de redes y se indican las herramientas disponibles.

**Procedimiento recomendado**: Automatización de la supervisión de la red remota con captura de paquetes.  
**Detalles**: supervise y diagnostique problemas de red sin iniciar sesión en las máquinas virtuales mediante Network Watcher. Desencadene la [captura de paquetes](../../network-watcher/network-watcher-alert-triggered-packet-capture.md) estableciendo alertas y obtenga acceso a información de rendimiento en tiempo real en el ámbito de paquete. Cuando vea un problema, podrá investigar en detalle para mejorar los diagnósticos.

**Procedimiento recomendado**: Extraer conclusiones sobre el tráfico de la red mediante registros de flujo.  
**Detalles**: conozca al detalle los patrones de tráfico de red mediante los [registros de flujo del grupo de seguridad de red](../../network-watcher/network-watcher-nsg-flow-logging-overview.md). La información de los registros de flujo le ayuda a recopilar datos para el cumplimiento, la auditoría y la supervisión del perfil de seguridad de red.

**Procedimiento recomendado**: diagnóstico de problemas de conectividad VPN.  
**Detalles**: use Network Watcher para [diagnosticar los problemas más comunes de conexión y VPN Gateway](../../network-watcher/network-watcher-diagnose-on-premises-connectivity.md). No solo puede identificar el problema, sino también usar registros detallados para investigar más.

## <a name="secure-deployment-by-using-proven-devops-tools"></a>Implementación segura mediante herramientas de DevOps comprobadas
Use los siguientes procedimientos recomendados de DevOps para garantizar que la empresa y los equipos sean productivos y eficientes.

**Procedimiento recomendado**: automatizar la compilación e implementación de servicios.  
**Detalles**: [Infraestructura como código](https://docs.microsoft.com/azure/devops/learn/what-is-infrastructure-as-code) es un conjunto de técnicas y procedimientos que ayudan a los profesionales de TI a eliminar la carga que supone la compilación y administración cotidianas de una infraestructura modular. Permite a los profesionales de TI compilar y mantener sus entornos de servidores modernos de forma similar a como los desarrolladores de software compilan y mantienen el código de las aplicaciones.

Puede usar [Azure Resource Manager](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/) para aprovisionar las aplicaciones mediante una plantilla declarativa. En una plantilla, puede implementar varios servicios junto con sus dependencias. Use la misma plantilla para implementar la aplicación de forma repetida durante cada fase de su ciclo de vida.

**Procedimiento recomendado**: compilar e implementar automáticamente en aplicaciones web o servicios en la nube de Azure.  
**Detalles**: Puede configurar Azure DevOps Projects para que se [compile e implemente de forma automática](https://docs.microsoft.com/azure/devops/pipelines/index?azure-devops) en aplicaciones web de Azure o en servicios en la nube. Azure DevOps implementa automáticamente los archivos binarios después de realizar una compilación en Azure tras cada comprobación de código. El proceso de compilación del paquete es equivalente al comando Package de Visual Studio y los pasos de publicación son equivalentes al comando Publish de Visual Studio.

**Procedimiento recomendado**: Automatice la administración de versiones.  
**Detalles**: [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/index?azure-devops) es una solución para automatizar la implementación en varias fases y la administración del proceso de publicación. Cree canalizaciones de implementación continua y administrada con el fin de que el lanzamiento sea rápido, sencillo y frecuente. Con Azure Pipelines, puede automatizar el proceso de publicación y tener flujos de trabajo de aprobación predefinidos. Realice la implementación de forma local y en la nube, amplíela y personalícela según sea necesario.

**Procedimiento recomendado**: Compruebe el rendimiento de su aplicación antes de iniciarla o de implementar actualizaciones en la producción.  
**Detalles**: Ejecute [pruebas de carga](/azure/devops/test/load-test/overview#alternatives) basadas en la nube para:

- Detectar problemas de rendimiento en la aplicación.
- Mejorar la calidad de implementación.
- Asegurarse de que la aplicación siempre está disponible.
- Asegurarse de que la aplicación puede controlar el tráfico de la siguiente campaña de marketing o el siguiente lanzamiento.

[Apache JMeter](https://jmeter.apache.org/) es una conocida herramienta gratuita de código abierto que cuenta con el respaldo de una sólida comunidad.

**Procedimiento recomendado**: supervisar el rendimiento de las aplicaciones.  
**Detalles**: [Azure Application Insights](../../azure-monitor/app/app-insights-overview.md) es un servicio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas. Use Application Insights para supervisar la aplicación web en vivo. Se detectan automáticamente las anomalías de rendimiento. Incluye herramientas de análisis que le ayudan a diagnosticar problemas y a comprender lo que hacen realmente los usuarios con la aplicación. Está diseñado para ayudarle a mejorar continuamente el rendimiento y la facilidad de uso.

## <a name="mitigate-and-protect-against-ddos"></a>Mitigar y proteger frente a DDoS
La denegación de servicio distribuido (DDoS) es un tipo de ataque que intenta agotar los recursos de la aplicación. El objetivo es afectar a la disponibilidad de la aplicación y a su capacidad para administrar solicitudes legítimas. Estos ataques son cada vez más sofisticados y tienen un mayor tamaño e impacto. Pueden ir dirigidos a cualquier punto de conexión que sea públicamente accesible a través de Internet.

Diseñar y compilar para la resistencia frente a DDoS requiere planear y diseñar para una serie de modos de error. Estos son los procedimientos recomendados para compilar servicios resistentes a DDoS en Azure.

**Procedimiento recomendado**: Asegúrese de que la seguridad es prioritaria durante todo el ciclo de vida de una aplicación, desde su diseño e implementación hasta la implementación y las operaciones. Las aplicaciones pueden tener errores que permiten que un volumen relativamente bajo de solicitudes usen muchos recursos y produzcan una interrupción del servicio.  
**Detalles**: Para proteger un servicio que se ejecuta en Microsoft Azure, debe conocer bien la arquitectura de su aplicación y centrarse en los [cinco pilares de la calidad del software](https://docs.microsoft.com/azure/architecture/guide/pillars). Debe conocer los volúmenes de tráfico típicos, el modelo de conectividad entre la aplicación y otras aplicaciones, y los puntos de conexión del servicio expuestos a la red pública de Internet.

Es de vital importancia garantizar que una aplicación sea lo suficientemente resistente para tratar con un ataque de denegación de servicio dirigido a la propia aplicación. La seguridad y la privacidad están integradas en la plataforma Azure, comenzando por el [ciclo de vida del desarrollo de la seguridad (SDL)](https://www.microsoft.com/sdl). El SDL aborda la seguridad en cada fase de desarrollo y se asegura de que Azure se actualice continuamente para que sea aún más seguro.

**Procedimiento recomendado**: Debe diseñar sus aplicaciones de modo que se puedan [escalar horizontalmente](https://docs.microsoft.com/azure/architecture/guide/design-principles/scale-out) para satisfacer la demanda de una carga mayor, específicamente en caso de un ataque de DDoS. Si la aplicación depende de una única instancia de un servicio, crea un único punto de error. El aprovisionamiento de varias instancias hace que el sistema sea más resistente y más escalable.  
**Detalles**: Para [Azure App Service](/azure/app-service/app-service-value-prop-what-is), seleccione un [Plan de App Service](../../app-service/overview-hosting-plans.md) que ofrezca varias instancias.

Para Azure Cloud Services, configure cada uno de los roles para utilizar [varias instancias](../../cloud-services/cloud-services-choose-me.md).

En el caso de [Azure Virtual Machines](../../virtual-machines/windows/overview.md), asegúrese de que la arquitectura de las máquinas virtuales incluya más de una máquina virtual y de que cada una de ellas se incluya en un [conjunto de disponibilidad](../../virtual-machines/windows/tutorial-availability-sets.md). Se recomienda usar conjuntos de escalado de máquinas virtuales para contar con funcionalidades de escalado automático.

**Procedimiento recomendado**: Disponer en niveles la defensa de la seguridad en una aplicación reduce las probabilidades de éxito de un ataque. Implemente diseños seguros para las aplicaciones con las funcionalidades integradas de la plataforma Azure.  
**Detalles**: el riesgo de ataque aumenta con el tamaño (área expuesta) de la aplicación. Puede reducir el área expuesta mediante la creación de listas blancas para cerrar el espacio de direcciones IP expuesto y los puertos de escucha que no sean necesarios en los equilibradores de carga ([Azure Load Balancer](/azure/load-balancer/load-balancer-get-started-internet-portal) y [Azure Application Gateway](/azure/application-gateway/application-gateway-create-probe-portal)).

Los [grupos de seguridad de red](../../virtual-network/security-overview.md) constituyen otra manera de reducir el área expuesta a ataques. Puede usar [etiquetas de servicio](../../virtual-network/security-overview.md#service-tags) y [grupos de seguridad de la aplicación](../../virtual-network/security-overview.md#application-security-groups) para minimizar la complejidad de la creación de reglas de seguridad y configurar la seguridad de la red como una extensión natural de la estructura de una aplicación.

Debe implementar los servicios de Azure en una [red virtual](../../virtual-network/virtual-networks-overview.md) siempre que sea posible. Este procedimiento permite que los recursos del servicio se comuniquen mediante direcciones IP privadas. De forma predeterminada, el tráfico de los servicios Azure desde una red virtual usa direcciones IP públicas como direcciones IP de origen.

Con los [puntos de conexión de servicio](../../virtual-network/virtual-network-service-endpoints-overview.md), el tráfico del servicio cambia para usar direcciones privadas de red virtual como direcciones IP de origen al acceder al servicio de Azure desde una red virtual.

Con frecuencia vemos ataques a los recursos locales de un cliente, además de a los recursos en Azure. Si conecta un entorno local a Azure, minimice la exposición de los recursos locales a la red pública de Internet.

Azure tiene dos [ofertas de servicio](../../virtual-network/ddos-protection-overview.md) de DDoS que proporcionan protección frente a ataques de red:

- La protección básica se integra en Azure de forma predeterminada sin costo adicional. El tamaño y la capacidad de la red de implementación global de Azure proporciona una defensa contra los ataques al nivel de red más comunes mediante la supervisión constante del tráfico y la mitigación en tiempo real. La protección básica no requiere ningún cambio de configuración ni de aplicación y ayuda a proteger todos los servicios de Azure, incluidos servicios PaaS como Azure DNS.
- La protección estándar proporciona funcionalidades avanzadas de mitigación de DDoS frente a ataques de red. Se ajusta automáticamente para proteger los recursos específicos de Azure. La protección se puede habilitar fácilmente durante la creación de redes virtuales. También puede realizarse después de la creación y no requiere ningún cambio de aplicación o recurso.

## <a name="enable-azure-policy"></a>Habilitación de Azure Policy
[Azure Policy](/azure/governance/policy/overview) es un servicio de Azure que se usa para crear, asignar y administrar directivas. Estas directivas aplican reglas y efectos a los recursos, con el fin de que estos sigan siendo compatibles con los estándares corporativos y los acuerdos de nivel de servicio. Azure Policy satisface esta necesidad mediante la evaluación de los recursos que incumplen las directivas asignadas.

Habilite Azure Policy para supervisar y aplicar la directiva escrita de la organización. Esto garantizará el cumplimiento de los requisitos de seguridad normativos o de la empresa mediante la administración centralizada de las directivas de seguridad en todas las cargas de trabajo en la nube híbrida. Más información sobre cómo [crear y administrar directivas para aplicar el cumplimiento](../../governance/policy/tutorials/create-and-manage.md). Vea [Estructura de definición de Azure Policy ](../../governance/policy/concepts/definition-structure.md) para obtener información general de los elementos de una directiva.

Estos son algunos procedimientos recomendados de seguridad que se pueden seguir tras la adopción de Azure Policy:

**Procedimiento recomendado**: La directiva admite varios tipos de efectos. Puede leer sobre ellos en [Estructura de definición de Azure Policy](../../governance/policy/concepts/definition-structure.md#policy-rule). Las operaciones empresariales se pueden ver afectadas negativamente por los efectos **deny** (denegar) y **remediate** (corregir), por lo que debe comenzar con el efecto **audit** (auditar) para limitar el riesgo de impacto negativo de la directiva.   
**Detalles**: [Inicie las implementaciones de directiva en modo audit](../../governance/policy/concepts/definition-structure.md#policy-rule) y, después, avance a **deny** o **remediate**. Pruebe y revise los resultados del efecto audit antes de pasar a **deny** o **remediate**.

Para más información, vea [Creación y administración de directivas para aplicar el cumplimiento](../../governance/policy/tutorials/create-and-manage.md).

**Procedimiento recomendado**: Identifique los roles responsables de supervisar las infracciones de directivas y garantizar que la acción correctora adecuada se realiza de forma rápida.   
**Detalles**: El rol asignado debe supervisar el cumplimiento normativo a través de [Azure Portal](../../governance/policy/how-to/get-compliance-data.md#portal) o la [línea de comandos](../../governance/policy/how-to/get-compliance-data.md#command-line).

**Procedimiento recomendado**: Azure Policy es una representación técnica de las directivas escritas de una organización. Para reducir la confusión y aumentar la coherencia, asigne todas las definiciones de Azure Policy a las directivas de la organización.   
**Detalles**: Para asignar documentos en la documentación de una organización o en la propia definición de Azure Policy, agregue una referencia a la directiva de la organización en la descripción de la [definición de la directiva](../../governance/policy/concepts/definition-structure.md#display-name-and-description) o de la [definición de la iniciativa](../../governance/policy/concepts/initiative-definition-structure.md#metadata).

## <a name="monitor-azure-ad-risk-reports"></a>Supervisión de los informes de riesgo de Azure AD
La mayoría de las infracciones de seguridad tienen lugar cuando los atacantes obtienen acceso a un entorno mediante el robo de identidad de un usuario. Descubrir las identidades en peligro no es tarea fácil. Azure AD emplea algoritmos y heurística de aprendizaje automático adaptable para detectar acciones sospechosas que están relacionadas con las cuentas de usuario. Cada acción sospechosa detectada se almacena en un registro llamado [detección de riesgos](../../active-directory/reports-monitoring/concept-risk-events.md). Las detecciones de riesgo se registran en los informes de seguridad de Azure AD. Para más información, vea el [informe de seguridad de usuarios en riesgo](../../active-directory/reports-monitoring/concept-user-at-risk.md) y el [informe de seguridad de inicios de sesión en riesgo](../../active-directory/reports-monitoring/concept-risky-sign-ins.md).

## <a name="next-steps"></a>Pasos siguientes
Vea [Patrones y procedimientos recomendados de seguridad en Azure](best-practices-and-patterns.md) para obtener más procedimientos recomendados de seguridad que pueda aplicar cuando diseñe, implemente y administre las soluciones en la nube mediante Azure.

En los siguientes recursos se ofrece más información general sobre la seguridad de Azure y los servicios de Microsoft relacionados:
* [Blog del equipo de seguridad de Azure](https://blogs.msdn.microsoft.com/azuresecurity/): ofrece información actualizada sobre lo último en seguridad de Azure
* [Microsoft Security Response Center](https://technet.microsoft.com/library/dn440717.aspx): aquí podrá notificar vulnerabilidades de seguridad de Microsoft, incluidos problemas con Azure, o también mediante correo electrónico a secure@microsoft.com.
